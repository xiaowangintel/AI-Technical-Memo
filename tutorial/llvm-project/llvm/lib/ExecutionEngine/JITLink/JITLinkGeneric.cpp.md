# JITLinkGeneric.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/JITLinkGeneric.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Generic JIT linker utilities.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===--------- JITLinkGeneric.cpp - Generic JIT linker utilities ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Generic JITLinker utility class.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 12-21
```cpp

#include "JITLinkGeneric.h"

#define DEBUG_TYPE "jitlink"

namespace llvm {
namespace jitlink {

JITLinkerBase::~JITLinkerBase() = default;

```
- **EN**: Pulls in the headers needed for this implementation, including `JITLinkGeneric.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `JITLinkGeneric.h`。

### Lines 22-34
```cpp
void JITLinkerBase::linkPhase1(std::unique_ptr<JITLinkerBase> Self) {

  LLVM_DEBUG(dbgs() << "Starting link phase 1\n");

  // Prune and optimize the graph.
  if (auto Err = runPasses(Passes.PrePrunePasses))
    return Ctx->notifyFailed(std::move(Err));

  LLVM_DEBUG({
    dbgs() << "Link graph pre-pruning:\n";
    G->dump(dbgs());
  });

```
- **EN**: Implements logic around `linkPhase1`, `notifyFailed`, `dbgs`, `dump`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `linkPhase1`, `notifyFailed`, `dbgs`, `dump` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 35-45
```cpp
  prune(*G);

  LLVM_DEBUG({
    dbgs() << "Link graph post-pruning:\n";
    G->dump(dbgs());
  });

  // Run post-pruning passes.
  if (auto Err = runPasses(Passes.PostPrunePasses))
    return Ctx->notifyFailed(std::move(Err));

```
- **EN**: Implements logic around `prune`, `dbgs`, `dump`, `notifyFailed`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `prune`, `dbgs`, `dump`, `notifyFailed` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 46-63
```cpp
  // Skip straight to phase 2 if the graph is empty with no associated actions.
  if (G->allocActions().empty() && llvm::all_of(G->sections(), [](Section &S) {
        return S.getMemLifetime() == orc::MemLifetime::NoAlloc;
      })) {
    linkPhase2(std::move(Self), nullptr);
    return;
  }

  Ctx->getMemoryManager().allocate(
      Ctx->getJITLinkDylib(), *G,
      [S = std::move(Self)](AllocResult AR) mutable {
        // FIXME: Once MSVC implements c++17 order of evaluation rules for calls
        // this can be simplified to
        //          S->linkPhase2(std::move(S), std::move(AR));
        auto *TmpSelf = S.get();
        TmpSelf->linkPhase2(std::move(S), std::move(AR));
      });
}
```
- **EN**: Implements logic around `getMemLifetime`, `linkPhase2`, `getMemoryManager`, `getJITLinkDylib`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getMemLifetime`, `linkPhase2`, `getMemoryManager`, `getJITLinkDylib`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 64-74
```cpp

void JITLinkerBase::linkPhase2(std::unique_ptr<JITLinkerBase> Self,
                               AllocResult AR) {

  LLVM_DEBUG(dbgs() << "Starting link phase 2\n");

  if (AR)
    Alloc = std::move(*AR);
  else
    return Ctx->notifyFailed(AR.takeError());

```
- **EN**: Implements logic around `linkPhase2`, `move`, `notifyFailed`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `linkPhase2`, `move`, `notifyFailed` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 75-83
```cpp
  LLVM_DEBUG({
    dbgs() << "Link graph before post-allocation passes:\n";
    G->dump(dbgs());
  });

  // Run post-allocation passes.
  if (auto Err = runPasses(Passes.PostAllocationPasses))
    return abandonAllocAndBailOut(std::move(Self), std::move(Err));

```
- **EN**: Implements logic around `dbgs`, `dump`, `abandonAllocAndBailOut`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `dump`, `abandonAllocAndBailOut` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 84-101
```cpp
  // Notify client that the defined symbols have been assigned addresses.
  LLVM_DEBUG(dbgs() << "Resolving symbols defined in " << G->getName() << "\n");

  if (auto Err = Ctx->notifyResolved(*G))
    return abandonAllocAndBailOut(std::move(Self), std::move(Err));

  auto ExternalSymbols = getExternalSymbolNames();

  // If there are no external symbols then proceed immediately with phase 3.
  if (ExternalSymbols.empty()) {
    LLVM_DEBUG({
      dbgs() << "No external symbols for " << G->getName()
             << ". Proceeding immediately with link phase 3.\n";
    });
    // FIXME: Once MSVC implements c++17 order of evaluation rules for calls
    // this can be simplified. See below.
    auto &TmpSelf = *Self;
    TmpSelf.linkPhase3(std::move(Self), AsyncLookupResult());
```
- **EN**: Implements logic around `abandonAllocAndBailOut`, `getExternalSymbolNames`, `dbgs`, `linkPhase3`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `abandonAllocAndBailOut`, `getExternalSymbolNames`, `dbgs`, `linkPhase3` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 102-110
```cpp
    return;
  }

  // Otherwise look up the externals.
  LLVM_DEBUG({
    dbgs() << "Issuing lookup for external symbols for " << G->getName()
           << " (may trigger materialization/linking of other graphs)...\n";
  });

```
- **EN**: Implements logic around `dbgs`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `dbgs` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 111-128
```cpp
  // We're about to hand off ownership of ourself to the continuation. Grab a
  // pointer to the context so that we can call it to initiate the lookup.
  //
  // FIXME: Once MSVC implements c++17 order of evaluation rules for calls this
  // can be simplified to:
  //
  // Ctx->lookup(std::move(UnresolvedExternals),
  //             [Self=std::move(Self)](Expected<AsyncLookupResult> Result) {
  //               Self->linkPhase3(std::move(Self), std::move(Result));
  //             });
  Ctx->lookup(std::move(ExternalSymbols),
              createLookupContinuation(
                  [S = std::move(Self)](
                      Expected<AsyncLookupResult> LookupResult) mutable {
                    auto &TmpSelf = *S;
                    TmpSelf.linkPhase3(std::move(S), std::move(LookupResult));
                  }));
}
```
- **EN**: Implements logic around `lookup`, `createLookupContinuation`, `move`, `linkPhase3`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `lookup`, `createLookupContinuation`, `move`, `linkPhase3` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 129-138
```cpp

void JITLinkerBase::linkPhase3(std::unique_ptr<JITLinkerBase> Self,
                               Expected<AsyncLookupResult> LR) {

  LLVM_DEBUG(dbgs() << "Starting link phase 3\n");

  // If the lookup failed, bail out.
  if (!LR)
    return abandonAllocAndBailOut(std::move(Self), LR.takeError());

```
- **EN**: Implements logic around `linkPhase3`, `abandonAllocAndBailOut`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `linkPhase3`, `abandonAllocAndBailOut` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 139-149
```cpp
  // Assign addresses to external addressables.
  applyLookupResult(*LR);

  LLVM_DEBUG({
    dbgs() << "Link graph before pre-fixup passes:\n";
    G->dump(dbgs());
  });

  if (auto Err = runPasses(Passes.PreFixupPasses))
    return abandonAllocAndBailOut(std::move(Self), std::move(Err));

```
- **EN**: Implements logic around `applyLookupResult`, `dbgs`, `dump`, `abandonAllocAndBailOut`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `applyLookupResult`, `dbgs`, `dump`, `abandonAllocAndBailOut` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 150-158
```cpp
  LLVM_DEBUG({
    dbgs() << "Link graph before copy-and-fixup:\n";
    G->dump(dbgs());
  });

  // Fix up block content.
  if (auto Err = fixUpBlocks(*G))
    return abandonAllocAndBailOut(std::move(Self), std::move(Err));

```
- **EN**: Implements logic around `dbgs`, `dump`, `abandonAllocAndBailOut`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `dump`, `abandonAllocAndBailOut` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 159-172
```cpp
  LLVM_DEBUG({
    dbgs() << "Link graph after copy-and-fixup:\n";
    G->dump(dbgs());
  });

  if (auto Err = runPasses(Passes.PostFixupPasses))
    return abandonAllocAndBailOut(std::move(Self), std::move(Err));

  // Skip straight to phase 4 if the graph has no allocation.
  if (!Alloc) {
    linkPhase4(std::move(Self), JITLinkMemoryManager::FinalizedAlloc{});
    return;
  }

```
- **EN**: Implements logic around `dbgs`, `dump`, `abandonAllocAndBailOut`, `linkPhase4`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `dump`, `abandonAllocAndBailOut`, `linkPhase4` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 173-181
```cpp
  Alloc->finalize([S = std::move(Self)](FinalizeResult FR) mutable {
    // FIXME: Once MSVC implements c++17 order of evaluation rules for calls
    // this can be simplified to
    //          S->linkPhase2(std::move(S), std::move(AR));
    auto *TmpSelf = S.get();
    TmpSelf->linkPhase4(std::move(S), std::move(FR));
  });
}

```
- **EN**: Implements logic around `finalize`, `get`, `linkPhase4`.
- **CN**: 围绕 `finalize`, `get`, `linkPhase4` 实现具体逻辑。

### Lines 182-191
```cpp
void JITLinkerBase::linkPhase4(std::unique_ptr<JITLinkerBase> Self,
                               FinalizeResult FR) {

  LLVM_DEBUG(dbgs() << "Starting link phase 4\n");

  if (!FR)
    return Ctx->notifyFailed(FR.takeError());

  Ctx->notifyFinalized(std::move(*FR));

```
- **EN**: Implements logic around `linkPhase4`, `notifyFailed`, `notifyFinalized`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `linkPhase4`, `notifyFailed`, `notifyFinalized` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 192-201
```cpp
  LLVM_DEBUG({ dbgs() << "Link complete\n"; });
}

Error JITLinkerBase::runPasses(LinkGraphPassList &Passes) {
  for (auto &P : Passes)
    if (auto Err = P(*G))
      return Err;
  return Error::success();
}

```
- **EN**: Implements logic around `runPasses`, `success`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `runPasses`, `success` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 202-216
```cpp
JITLinkContext::LookupMap JITLinkerBase::getExternalSymbolNames() const {
  // Identify unresolved external symbols.
  JITLinkContext::LookupMap UnresolvedExternals;
  for (auto *Sym : G->external_symbols()) {
    assert(!Sym->getAddress() &&
           "External has already been assigned an address");
    assert(Sym->hasName() && "Externals must be named");
    SymbolLookupFlags LookupFlags =
        Sym->isWeaklyReferenced() ? SymbolLookupFlags::WeaklyReferencedSymbol
                                  : SymbolLookupFlags::RequiredSymbol;
    UnresolvedExternals[Sym->getName()] = LookupFlags;
  }
  return UnresolvedExternals;
}

```
- **EN**: Implements logic around `getExternalSymbolNames`, `assert`, `isWeaklyReferenced`, `getName`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getExternalSymbolNames`, `assert`, `isWeaklyReferenced`, `getName` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 217-234
```cpp
void JITLinkerBase::applyLookupResult(AsyncLookupResult Result) {
  for (auto *Sym : G->external_symbols()) {
    assert(Sym->getOffset() == 0 &&
           "External symbol is not at the start of its addressable block");
    assert(!Sym->getAddress() && "Symbol already resolved");
    assert(!Sym->isDefined() && "Symbol being resolved is already defined");
    auto ResultI = Result.find(Sym->getName());
    if (ResultI != Result.end()) {
      Sym->getAddressable().setAddress(ResultI->second.getAddress());
      Sym->setLinkage(ResultI->second.getFlags().isWeak() ? Linkage::Weak
                                                          : Linkage::Strong);
      Sym->setScope(ResultI->second.getFlags().isExported() ? Scope::Default
                                                            : Scope::Hidden);
    } else
      assert(Sym->isWeaklyReferenced() &&
             "Failed to resolve non-weak reference");
  }

```
- **EN**: Implements logic around `applyLookupResult`, `assert`, `find`, `getAddressable`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `applyLookupResult`, `assert`, `find`, `getAddressable`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 235-252
```cpp
  LLVM_DEBUG({
    dbgs() << "Externals after applying lookup result:\n";
    for (auto *Sym : G->external_symbols()) {
      dbgs() << "  " << Sym->getName() << ": "
             << formatv("{0:x16}", Sym->getAddress().getValue());
      switch (Sym->getLinkage()) {
      case Linkage::Strong:
        break;
      case Linkage::Weak:
        dbgs() << " (weak)";
        break;
      }
      switch (Sym->getScope()) {
      case Scope::Local:
      case Scope::SideEffectsOnly:
        llvm_unreachable("External symbol should not have local or "
                         "side-effects-only linkage");
      case Scope::Hidden:
```
- **EN**: Implements logic around `dbgs`, `formatv`, `llvm_unreachable`; this block uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `dbgs`, `formatv`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 253-262
```cpp
        break;
      case Scope::Default:
        dbgs() << " (exported)";
        break;
      }
      dbgs() << "\n";
    }
  });
}

```
- **EN**: Implements logic around `dbgs`.
- **CN**: 围绕 `dbgs` 实现具体逻辑。

### Lines 263-271
```cpp
void JITLinkerBase::abandonAllocAndBailOut(std::unique_ptr<JITLinkerBase> Self,
                                           Error Err) {
  assert(Err && "Should not be bailing out on success value");
  assert(Alloc && "can not call abandonAllocAndBailOut before allocation");
  Alloc->abandon([S = std::move(Self), E1 = std::move(Err)](Error E2) mutable {
    S->Ctx->notifyFailed(joinErrors(std::move(E1), std::move(E2)));
  });
}

```
- **EN**: Implements logic around `abandonAllocAndBailOut`, `assert`, `abandon`, `notifyFailed`.
- **CN**: 围绕 `abandonAllocAndBailOut`, `assert`, `abandon`, `notifyFailed` 实现具体逻辑。

### Lines 272-280
```cpp
void prune(LinkGraph &G) {
  std::vector<Symbol *> Worklist;
  DenseSet<Block *> VisitedBlocks;

  // Build the initial worklist from all symbols initially live.
  for (auto *Sym : G.defined_symbols())
    if (Sym->isLive())
      Worklist.push_back(Sym);

```
- **EN**: Implements logic around `prune`, `push_back`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `prune`, `push_back` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 281-291
```cpp
  // Propagate live flags to all symbols reachable from the initial live set.
  while (!Worklist.empty()) {
    auto *Sym = Worklist.back();
    Worklist.pop_back();

    auto &B = Sym->getBlock();

    // Skip addressables that we've visited before.
    if (VisitedBlocks.count(&B))
      continue;

```
- **EN**: Implements logic around `back`, `pop_back`, `getBlock`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `back`, `pop_back`, `getBlock` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 292-304
```cpp
    VisitedBlocks.insert(&B);

    for (auto &E : Sym->getBlock().edges()) {
      // If the edge target is a defined symbol that is being newly marked live
      // then add it to the worklist.
      if (E.getTarget().isDefined() && !E.getTarget().isLive())
        Worklist.push_back(&E.getTarget());

      // Mark the target live.
      E.getTarget().setLive(true);
    }
  }

```
- **EN**: Implements logic around `insert`, `push_back`, `getTarget`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `insert`, `push_back`, `getTarget` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 305-317
```cpp
  // Collect all defined symbols to remove, then remove them.
  {
    LLVM_DEBUG(dbgs() << "Dead-stripping defined symbols:\n");
    std::vector<Symbol *> SymbolsToRemove;
    for (auto *Sym : G.defined_symbols())
      if (!Sym->isLive())
        SymbolsToRemove.push_back(Sym);
    for (auto *Sym : SymbolsToRemove) {
      LLVM_DEBUG(dbgs() << "  " << *Sym << "...\n");
      G.removeDefinedSymbol(*Sym);
    }
  }

```
- **EN**: Implements logic around `push_back`, `removeDefinedSymbol`.
- **CN**: 围绕 `push_back`, `removeDefinedSymbol` 实现具体逻辑。

### Lines 318-330
```cpp
  // Delete any unused blocks.
  {
    LLVM_DEBUG(dbgs() << "Dead-stripping blocks:\n");
    std::vector<Block *> BlocksToRemove;
    for (auto *B : G.blocks())
      if (!VisitedBlocks.count(B))
        BlocksToRemove.push_back(B);
    for (auto *B : BlocksToRemove) {
      LLVM_DEBUG(dbgs() << "  " << *B << "...\n");
      G.removeBlock(*B);
    }
  }

```
- **EN**: Implements logic around `push_back`, `removeBlock`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `push_back`, `removeBlock` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 331-344
```cpp
  // Collect all external symbols to remove, then remove them.
  {
    LLVM_DEBUG(dbgs() << "Removing unused external symbols:\n");
    std::vector<Symbol *> SymbolsToRemove;
    for (auto *Sym : G.external_symbols())
      if (!Sym->isLive())
        SymbolsToRemove.push_back(Sym);
    for (auto *Sym : SymbolsToRemove) {
      LLVM_DEBUG(dbgs() << "  " << *Sym << "...\n");
      G.removeExternalSymbol(*Sym);
    }
  }
}

```
- **EN**: Implements logic around `push_back`, `removeExternalSymbol`.
- **CN**: 围绕 `push_back`, `removeExternalSymbol` 实现具体逻辑。

### Lines 345-346
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
- **Symbol materialization / 符号物化**:
  - **EN**: Defers code or data generation until symbols are requested
  - **CN**: 将代码或数据的生成延迟到符号真正被请求时
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

- **Direct includes / 直接包含**: `JITLinkGeneric.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
