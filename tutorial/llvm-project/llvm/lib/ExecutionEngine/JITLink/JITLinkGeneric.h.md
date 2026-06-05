# JITLinkGeneric.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/JITLinkGeneric.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Generic JIT linker utilities.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===------ JITLinkGeneric.h - Generic JIT linker utilities -----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-15
```cpp
//
// Generic JITLinker utilities. E.g. graph pruning, eh-frame parsing.
//
//===----------------------------------------------------------------------===//

#ifndef LIB_EXECUTIONENGINE_JITLINK_JITLINKGENERIC_H
#define LIB_EXECUTIONENGINE_JITLINK_JITLINKGENERIC_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 16-22
```cpp
#include "llvm/ExecutionEngine/JITLink/JITLink.h"

#define DEBUG_TYPE "jitlink"

namespace llvm {
namespace jitlink {

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/JITLink/JITLink.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/JITLink/JITLink.h`。

### Lines 23-36
```cpp
/// Base class for a JIT linker.
///
/// A JITLinkerBase instance links one object file into an ongoing JIT
/// session. Symbol resolution and finalization operations are pluggable,
/// and called using continuation passing (passing a continuation for the
/// remaining linker work) to allow them to be performed asynchronously.
class JITLinkerBase {
public:
  JITLinkerBase(std::unique_ptr<JITLinkContext> Ctx,
                std::unique_ptr<LinkGraph> G, PassConfiguration Passes)
      : Ctx(std::move(Ctx)), G(std::move(G)), Passes(std::move(Passes)) {
    assert(this->Ctx && "Ctx can not be null");
    assert(this->G && "G can not be null");
  }
```
- **EN**: Introduces declarations for `for`, `JITLinkerBase`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `for`, `JITLinkerBase` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 37-44
```cpp

  virtual ~JITLinkerBase();

protected:
  using InFlightAlloc = JITLinkMemoryManager::InFlightAlloc;
  using AllocResult = Expected<std::unique_ptr<InFlightAlloc>>;
  using FinalizeResult = Expected<JITLinkMemoryManager::FinalizedAlloc>;

```
- **EN**: Implements logic around `~JITLinkerBase`.
- **CN**: 围绕 `~JITLinkerBase` 实现具体逻辑。

### Lines 45-54
```cpp
  // Returns a reference to the graph being linked.
  LinkGraph &getGraph() { return *G; }

  // Returns true if the context says that the linker should add default
  // passes. This can be used by JITLinkerBase implementations when deciding
  // whether they should add default passes.
  bool shouldAddDefaultTargetPasses(const Triple &TT) {
    return Ctx->shouldAddDefaultTargetPasses(TT);
  }

```
- **EN**: Implements logic around `getGraph`, `shouldAddDefaultTargetPasses`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getGraph`, `shouldAddDefaultTargetPasses` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 55-67
```cpp
  // Returns the PassConfiguration for this instance. This can be used by
  // JITLinkerBase implementations to add late passes that reference their
  // own data structures (e.g. for ELF implementations to locate / construct
  // a GOT start symbol prior to fixup).
  PassConfiguration &getPassConfig() { return Passes; }

  // Phase 1:
  //   1.1: Run pre-prune passes
  //   1.2: Prune graph
  //   1.3: Run post-prune passes
  //   1.4: Allocate memory.
  void linkPhase1(std::unique_ptr<JITLinkerBase> Self);

```
- **EN**: Implements logic around `getPassConfig`, `linkPhase1`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getPassConfig`, `linkPhase1` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 68-81
```cpp
  // Phase 2:
  //   2.2: Run post-allocation passes
  //   2.3: Notify context of final assigned symbol addresses
  //   2.4: Identify external symbols and make an async call to resolve
  void linkPhase2(std::unique_ptr<JITLinkerBase> Self, AllocResult AR);

  // Phase 3:
  //   3.1: Apply resolution results
  //   3.2: Run pre-fixup passes
  //   3.3: Fix up block contents
  //   3.4: Run post-fixup passes
  //   3.5: Make an async call to transfer and finalize memory.
  void linkPhase3(std::unique_ptr<JITLinkerBase> Self,
                  Expected<AsyncLookupResult> LookupResult);
```
- **EN**: Implements logic around `linkPhase2`, `linkPhase3`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `linkPhase2`, `linkPhase3` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 82-91
```cpp

  // Phase 4:
  //   4.1: Call OnFinalized callback, handing off allocation.
  void linkPhase4(std::unique_ptr<JITLinkerBase> Self, FinalizeResult FR);

private:
  // Run all passes in the given pass list, bailing out immediately if any pass
  // returns an error.
  Error runPasses(LinkGraphPassList &Passes);

```
- **EN**: Implements logic around `linkPhase4`, `runPasses`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `linkPhase4`, `runPasses` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 92-99
```cpp
  // Copy block contents and apply relocations.
  // Implemented in JITLinker.
  virtual Error fixUpBlocks(LinkGraph &G) const = 0;

  JITLinkContext::LookupMap getExternalSymbolNames() const;
  void applyLookupResult(AsyncLookupResult LR);
  void abandonAllocAndBailOut(std::unique_ptr<JITLinkerBase> Self, Error Err);

```
- **EN**: Implements logic around `fixUpBlocks`, `getExternalSymbolNames`, `applyLookupResult`, `abandonAllocAndBailOut`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `fixUpBlocks`, `getExternalSymbolNames`, `applyLookupResult`, `abandonAllocAndBailOut` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 100-109
```cpp
  std::unique_ptr<JITLinkContext> Ctx;
  std::unique_ptr<LinkGraph> G;
  PassConfiguration Passes;
  std::unique_ptr<InFlightAlloc> Alloc;
};

template <typename LinkerImpl> class JITLinker : public JITLinkerBase {
public:
  using JITLinkerBase::JITLinkerBase;

```
- **EN**: Introduces declarations for `JITLinker`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `JITLinker` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 110-123
```cpp
  /// Link constructs a LinkerImpl instance and calls linkPhase1.
  /// Link should be called with the constructor arguments for LinkerImpl, which
  /// will be forwarded to the constructor.
  template <typename... ArgTs> static void link(ArgTs &&... Args) {
    auto L = std::make_unique<LinkerImpl>(std::forward<ArgTs>(Args)...);

    // Ownership of the linker is passed into the linker's doLink function to
    // allow it to be passed on to async continuations.
    //
    // FIXME: Remove LTmp once we have c++17.
    // C++17 sequencing rules guarantee that function name expressions are
    // sequenced before arguments, so L->linkPhase1(std::move(L), ...) will be
    // well formed.
    auto &LTmp = *L;
```
- **EN**: Implements logic around `link`, `make_unique<LinkerImpl>`.
- **CN**: 围绕 `link`, `make_unique<LinkerImpl>` 实现具体逻辑。

### Lines 124-131
```cpp
    LTmp.linkPhase1(std::move(L));
  }

private:
  const LinkerImpl &impl() const {
    return static_cast<const LinkerImpl &>(*this);
  }

```
- **EN**: Implements logic around `linkPhase1`, `impl`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `linkPhase1`, `impl` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 132-140
```cpp
  Error fixUpBlocks(LinkGraph &G) const override {
    LLVM_DEBUG(dbgs() << "Fixing up blocks:\n");

    for (auto &Sec : G.sections()) {
      bool NoAllocSection = Sec.getMemLifetime() == orc::MemLifetime::NoAlloc;

      for (auto *B : Sec.blocks()) {
        LLVM_DEBUG(dbgs() << "  " << *B << ":\n");

```
- **EN**: Implements logic around `fixUpBlocks`, `getMemLifetime`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `fixUpBlocks`, `getMemLifetime` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 141-149
```cpp
        // Copy Block data and apply fixups.
        LLVM_DEBUG(dbgs() << "    Applying fixups.\n");
        assert((!B->isZeroFill() || all_of(B->edges(),
                                           [](const Edge &E) {
                                             return E.getKind() ==
                                                    Edge::KeepAlive;
                                           })) &&
               "Non-KeepAlive edges in zero-fill block?");

```
- **EN**: Implements logic around `assert`, `getKind`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `getKind` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 150-157
```cpp
        // If this is a no-alloc section then copy the block content into
        // memory allocated on the Graph's allocator (if it hasn't been
        // already).
        if (NoAllocSection)
          (void)B->getMutableContent(G);

        for (auto &E : B->edges()) {

```
- **EN**: Implements logic around `getMutableContent`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `getMutableContent` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 158-169
```cpp
          // Skip non-relocation edges.
          if (!E.isRelocation())
            continue;

          // If B is a block in a Standard or Finalize section then make sure
          // that no edges point to symbols in NoAlloc sections.
          assert((NoAllocSection || !E.getTarget().isDefined() ||
                  E.getTarget().getSection().getMemLifetime() !=
                      orc::MemLifetime::NoAlloc) &&
                 "Block in allocated section has edge pointing to no-alloc "
                 "section");

```
- **EN**: Implements logic around `assert`, `getTarget`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `assert`, `getTarget` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 170-176
```cpp
          // Dispatch to LinkerImpl for fixup.
          if (auto Err = impl().applyFixup(G, *B, E))
            return Err;
        }
      }
    }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 177-183
```cpp
    return Error::success();
  }
};

/// Removes dead symbols/blocks/addressables.
///
/// Finds the set of symbols and addressables reachable from any symbol
```
- **EN**: Implements logic around `success`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `success` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 184-190
```cpp
/// initially marked live. All symbols/addressables not marked live at the end
/// of this process are removed.
void prune(LinkGraph &G);

} // end namespace jitlink
} // end namespace llvm

```
- **EN**: Introduces declarations for `jitlink`, `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `jitlink`, `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 191-193
```cpp
#undef DEBUG_TYPE // "jitlink"

#endif // LIB_EXECUTIONENGINE_JITLINK_JITLINKGENERIC_H
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

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

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/JITLink/JITLink.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
