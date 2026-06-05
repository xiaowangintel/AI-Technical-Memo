# IRPartitionLayer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/IRPartitionLayer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Partition IR module into submodules.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===----- IRPartitionLayer.cpp - Partition IR module into submodules -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ExecutionEngine/Orc/IRPartitionLayer.h"
#include "llvm/ExecutionEngine/Orc/ExecutionUtils.h"
#include "llvm/ExecutionEngine/Orc/IndirectionUtils.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/IRPartitionLayer.h`, `llvm/ExecutionEngine/Orc/ExecutionUtils.h`, `llvm/ExecutionEngine/Orc/IndirectionUtils.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/IRPartitionLayer.h`, `llvm/ExecutionEngine/Orc/ExecutionUtils.h`, `llvm/ExecutionEngine/Orc/IndirectionUtils.h`。

### Lines 13-23
```cpp
using namespace llvm;
using namespace llvm::orc;

static ThreadSafeModule extractSubModule(ThreadSafeModule &TSM,
                                         StringRef Suffix,
                                         GVPredicate ShouldExtract) {

  auto DeleteExtractedDefs = [](GlobalValue &GV) {
    // Bump the linkage: this global will be provided by the external module.
    GV.setLinkage(GlobalValue::ExternalLinkage);

```
- **EN**: Introduces declarations for `llvm`, `llvm::orc`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `llvm::orc` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 24-39
```cpp
    // Delete the definition in the source module.
    if (isa<Function>(GV)) {
      auto &F = cast<Function>(GV);
      F.deleteBody();
      F.setPersonalityFn(nullptr);
    } else if (isa<GlobalVariable>(GV)) {
      cast<GlobalVariable>(GV).setInitializer(nullptr);
    } else if (isa<GlobalAlias>(GV)) {
      // We need to turn deleted aliases into function or variable decls based
      // on the type of their aliasee.
      auto &A = cast<GlobalAlias>(GV);
      Constant *Aliasee = A.getAliasee();
      assert(A.hasName() && "Anonymous alias?");
      assert(Aliasee->hasName() && "Anonymous aliasee");
      std::string AliasName = std::string(A.getName());

```
- **EN**: Implements logic around `cast<Function>`, `deleteBody`, `setPersonalityFn`, `cast<GlobalVariable>`, and 4 more symbols.
- **CN**: 围绕 `cast<Function>`, `deleteBody`, `setPersonalityFn`, `cast<GlobalVariable>`, and 4 more symbols 实现具体逻辑。

### Lines 40-56
```cpp
      if (isa<Function>(Aliasee)) {
        auto *F = cloneFunctionDecl(*A.getParent(), *cast<Function>(Aliasee));
        A.replaceAllUsesWith(F);
        A.eraseFromParent();
        F->setName(AliasName);
      } else if (isa<GlobalVariable>(Aliasee)) {
        auto *G = cloneGlobalVariableDecl(*A.getParent(),
                                          *cast<GlobalVariable>(Aliasee));
        A.replaceAllUsesWith(G);
        A.eraseFromParent();
        G->setName(AliasName);
      } else
        llvm_unreachable("Alias to unsupported type");
    } else
      llvm_unreachable("Unsupported global type");
  };

```
- **EN**: Implements logic around `cloneFunctionDecl`, `replaceAllUsesWith`, `eraseFromParent`, `setName`, and 3 more symbols.
- **CN**: 围绕 `cloneFunctionDecl`, `replaceAllUsesWith`, `eraseFromParent`, `setName`, and 3 more symbols 实现具体逻辑。

### Lines 57-67
```cpp
  auto NewTSM = cloneToNewContext(TSM, ShouldExtract, DeleteExtractedDefs);
  NewTSM.withModuleDo([&](Module &M) {
    M.setModuleIdentifier((M.getModuleIdentifier() + Suffix).str());
  });

  return NewTSM;
}

namespace llvm {
namespace orc {

```
- **EN**: Introduces declarations for `llvm`, `orc`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `orc` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 68-82
```cpp
class PartitioningIRMaterializationUnit : public IRMaterializationUnit {
public:
  PartitioningIRMaterializationUnit(ExecutionSession &ES,
                                    const IRSymbolMapper::ManglingOptions &MO,
                                    ThreadSafeModule TSM,
                                    IRPartitionLayer &Parent)
      : IRMaterializationUnit(ES, MO, std::move(TSM)), Parent(Parent) {}

  PartitioningIRMaterializationUnit(
      ThreadSafeModule TSM, Interface I,
      SymbolNameToDefinitionMap SymbolToDefinition, IRPartitionLayer &Parent)
      : IRMaterializationUnit(std::move(TSM), std::move(I),
                              std::move(SymbolToDefinition)),
        Parent(Parent) {}

```
- **EN**: Introduces declarations for `PartitioningIRMaterializationUnit`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `PartitioningIRMaterializationUnit` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 83-95
```cpp
private:
  void materialize(std::unique_ptr<MaterializationResponsibility> R) override {
    Parent.emitPartition(std::move(R), std::move(TSM),
                         std::move(SymbolToDefinition));
  }

  void discard(const JITDylib &V, const SymbolStringPtr &Name) override {
    // All original symbols were materialized by the CODLayer and should be
    // final. The function bodies provided by M should never be overridden.
    llvm_unreachable("Discard should never be called on an "
                     "ExtractingIRMaterializationUnit");
  }

```
- **EN**: Implements logic around `materialize`, `emitPartition`, `move`, `discard`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `materialize`, `emitPartition`, `move`, `discard`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 96-104
```cpp
  IRPartitionLayer &Parent;
};

} // namespace orc
} // namespace llvm

IRPartitionLayer::IRPartitionLayer(ExecutionSession &ES, IRLayer &BaseLayer)
    : IRLayer(ES, BaseLayer.getManglingOptions()), BaseLayer(BaseLayer) {}

```
- **EN**: Introduces declarations for `orc`, `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `orc`, `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 105-113
```cpp
void IRPartitionLayer::setPartitionFunction(PartitionFunction Partition) {
  this->Partition = Partition;
}

std::optional<IRPartitionLayer::GlobalValueSet>
IRPartitionLayer::compileRequested(GlobalValueSet Requested) {
  return std::move(Requested);
}

```
- **EN**: Implements logic around `setPartitionFunction`, `compileRequested`, `move`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `setPartitionFunction`, `compileRequested`, `move` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 114-122
```cpp
std::optional<IRPartitionLayer::GlobalValueSet>
IRPartitionLayer::compileWholeModule(GlobalValueSet Requested) {
  return std::nullopt;
}

void IRPartitionLayer::emit(std::unique_ptr<MaterializationResponsibility> R,
                            ThreadSafeModule TSM) {
  assert(TSM && "Null module");

```
- **EN**: Implements logic around `compileWholeModule`, `emit`, `assert`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `compileWholeModule`, `emit`, `assert` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 123-137
```cpp
  auto &ES = getExecutionSession();
  TSM.withModuleDo([&](Module &M) {
    // First, do some cleanup on the module:
    cleanUpModule(M);
  });

  // Create a partitioning materialization unit and pass the responsibility.
  if (auto Err = R->replace(std::make_unique<PartitioningIRMaterializationUnit>(
          ES, *getManglingOptions(), std::move(TSM), *this))) {
    ES.reportError(std::move(Err));
    R->failMaterialization();
    return;
  }
}

```
- **EN**: Implements logic around `getExecutionSession`, `withModuleDo`, `cleanUpModule`, `getManglingOptions`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `getExecutionSession`, `withModuleDo`, `cleanUpModule`, `getManglingOptions`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 138-150
```cpp
void IRPartitionLayer::cleanUpModule(Module &M) {
  for (auto &F : M.functions()) {
    if (F.isDeclaration())
      continue;

    if (F.hasAvailableExternallyLinkage()) {
      F.deleteBody();
      F.setPersonalityFn(nullptr);
      continue;
    }
  }
}

```
- **EN**: Implements logic around `cleanUpModule`, `deleteBody`, `setPersonalityFn`.
- **CN**: 围绕 `cleanUpModule`, `deleteBody`, `setPersonalityFn` 实现具体逻辑。

### Lines 151-159
```cpp
void IRPartitionLayer::expandPartition(GlobalValueSet &Partition) {
  // Expands the partition to ensure the following rules hold:
  // (1) If any alias is in the partition, its aliasee is also in the partition.
  // (2) If any aliasee is in the partition, its aliases are also in the
  //     partiton.
  // (3) If any global variable is in the partition then all global variables
  //     are in the partition.
  assert(!Partition.empty() && "Unexpected empty partition");

```
- **EN**: Implements logic around `expandPartition`, `assert`.
- **CN**: 围绕 `expandPartition`, `assert` 实现具体逻辑。

### Lines 160-170
```cpp
  const Module &M = *(*Partition.begin())->getParent();
  bool ContainsGlobalVariables = false;
  std::vector<const GlobalValue *> GVsToAdd;

  for (const auto *GV : Partition)
    if (isa<GlobalAlias>(GV))
      GVsToAdd.push_back(
          cast<GlobalValue>(cast<GlobalAlias>(GV)->getAliasee()));
    else if (isa<GlobalVariable>(GV))
      ContainsGlobalVariables = true;

```
- **EN**: Implements logic around `begin`, `push_back`, `cast<GlobalValue>`.
- **CN**: 围绕 `begin`, `push_back`, `cast<GlobalValue>` 实现具体逻辑。

### Lines 171-182
```cpp
  for (auto &A : M.aliases())
    if (Partition.count(cast<GlobalValue>(A.getAliasee())))
      GVsToAdd.push_back(&A);

  if (ContainsGlobalVariables)
    for (auto &G : M.globals())
      GVsToAdd.push_back(&G);

  for (const auto *GV : GVsToAdd)
    Partition.insert(GV);
}

```
- **EN**: Implements logic around `push_back`, `insert`.
- **CN**: 围绕 `push_back`, `insert` 实现具体逻辑。

### Lines 183-191
```cpp
void IRPartitionLayer::emitPartition(
    std::unique_ptr<MaterializationResponsibility> R, ThreadSafeModule TSM,
    IRMaterializationUnit::SymbolNameToDefinitionMap Defs) {

  // FIXME: Need a 'notify lazy-extracting/emitting' callback to tie the
  //        extracted module key, extracted module, and source module key
  //        together. This could be used, for example, to provide a specific
  //        memory manager instance to the linking layer.

```
- **EN**: Implements logic around `emitPartition`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `emitPartition` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 192-205
```cpp
  auto &ES = getExecutionSession();
  GlobalValueSet RequestedGVs;
  for (auto &Name : R->getRequestedSymbols()) {
    if (Name == R->getInitializerSymbol())
      TSM.withModuleDo([&](Module &M) {
        for (auto &GV : getStaticInitGVs(M))
          RequestedGVs.insert(&GV);
      });
    else {
      assert(Defs.count(Name) && "No definition for symbol");
      RequestedGVs.insert(Defs[Name]);
    }
  }

```
- **EN**: Implements logic around `getExecutionSession`, `withModuleDo`, `insert`, `assert`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `getExecutionSession`, `withModuleDo`, `insert`, `assert` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 206-219
```cpp
  /// Perform partitioning with the context lock held, since the partition
  /// function is allowed to access the globals to compute the partition.
  auto GVsToExtract =
      TSM.withModuleDo([&](Module &M) { return Partition(RequestedGVs); });

  // Take a 'None' partition to mean the whole module (as opposed to an empty
  // partition, which means "materialize nothing"). Emit the whole module
  // unmodified to the base layer.
  if (GVsToExtract == std::nullopt) {
    Defs.clear();
    BaseLayer.emit(std::move(R), std::move(TSM));
    return;
  }

```
- **EN**: Implements logic around `withModuleDo`, `clear`, `emit`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `withModuleDo`, `clear`, `emit` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 220-234
```cpp
  // If the partition is empty, return the whole module to the symbol table.
  if (GVsToExtract->empty()) {
    if (auto Err =
            R->replace(std::make_unique<PartitioningIRMaterializationUnit>(
                std::move(TSM),
                MaterializationUnit::Interface(R->getSymbols(),
                                               R->getInitializerSymbol()),
                std::move(Defs), *this))) {
      getExecutionSession().reportError(std::move(Err));
      R->failMaterialization();
      return;
    }
    return;
  }

```
- **EN**: Implements logic around `replace`, `move`, `Interface`, `getInitializerSymbol`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `replace`, `move`, `Interface`, `getInitializerSymbol`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 235-246
```cpp
  // Ok -- we actually need to partition the symbols. Promote the symbol
  // linkages/names, expand the partition to include any required symbols
  // (i.e. symbols that can't be separated from our partition), and
  // then extract the partition.
  //
  // FIXME: We apply this promotion once per partitioning. It's safe, but
  // overkill.
  auto ExtractedTSM = TSM.withModuleDo([&](Module &M)
                                           -> Expected<ThreadSafeModule> {
    auto PromotedGlobals = PromoteSymbols(M);
    if (!PromotedGlobals.empty()) {

```
- **EN**: Implements logic around `withModuleDo`, `PromoteSymbols`.
- **CN**: 围绕 `withModuleDo`, `PromoteSymbols` 实现具体逻辑。

### Lines 247-255
```cpp
      MangleAndInterner Mangle(ES, M.getDataLayout());
      SymbolFlagsMap SymbolFlags;
      IRSymbolMapper::add(ES, *getManglingOptions(), PromotedGlobals,
                          SymbolFlags);

      if (auto Err = R->defineMaterializing(SymbolFlags))
        return std::move(Err);
    }

```
- **EN**: Implements logic around `Mangle`, `add`, `move`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `Mangle`, `add`, `move` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 256-273
```cpp
    expandPartition(*GVsToExtract);

    // Submodule name is given by hashing the names of the globals.
    std::string SubModuleName;
    {
      std::vector<const GlobalValue *> HashGVs;
      HashGVs.reserve(GVsToExtract->size());
      llvm::append_range(HashGVs, *GVsToExtract);
      llvm::sort(HashGVs, [](const GlobalValue *LHS, const GlobalValue *RHS) {
        return LHS->getName() < RHS->getName();
      });
      hash_code HC(0);
      for (const auto *GV : HashGVs) {
        assert(GV->hasName() && "All GVs to extract should be named by now");
        auto GVName = GV->getName();
        HC = hash_combine(HC, hash_combine_range(GVName));
      }
      raw_string_ostream(SubModuleName)
```
- **EN**: Implements logic around `expandPartition`, `reserve`, `append_range`, `sort`, and 5 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `expandPartition`, `reserve`, `append_range`, `sort`, and 5 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 274-285
```cpp
          << ".submodule."
          << formatv(sizeof(size_t) == 8 ? "{0:x16}" : "{0:x8}",
                     static_cast<size_t>(HC))
          << ".ll";
    }

    // Extract the requested partiton (plus any necessary aliases) and
    // put the rest back into the impl dylib.
    auto ShouldExtract = [&](const GlobalValue &GV) -> bool {
      return GVsToExtract->count(&GV);
    };

```
- **EN**: Implements logic around `formatv`, `static_cast<size_t>`, `count`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `formatv`, `static_cast<size_t>`, `count` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 286-294
```cpp
    return extractSubModule(TSM, SubModuleName, ShouldExtract);
  });

  if (!ExtractedTSM) {
    ES.reportError(ExtractedTSM.takeError());
    R->failMaterialization();
    return;
  }

```
- **EN**: Implements logic around `extractSubModule`, `reportError`, `failMaterialization`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `extractSubModule`, `reportError`, `failMaterialization` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 295-302
```cpp
  if (auto Err = R->replace(std::make_unique<PartitioningIRMaterializationUnit>(
          ES, *getManglingOptions(), std::move(TSM), *this))) {
    ES.reportError(std::move(Err));
    R->failMaterialization();
    return;
  }
  BaseLayer.emit(std::move(R), std::move(*ExtractedTSM));
}
```
- **EN**: Implements logic around `getManglingOptions`, `reportError`, `failMaterialization`, `emit`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `getManglingOptions`, `reportError`, `failMaterialization`, `emit` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

## Key Concepts / 关键概念

- **ORC JIT / ORC JIT**:
  - **EN**: Coordinates JITDylibs, symbol materialization, execution sessions, and asynchronous compilation flows
  - **CN**: 协调 JITDylib、符号物化、ExecutionSession 与异步编译流程
- **Symbol materialization / 符号物化**:
  - **EN**: Defers code or data generation until symbols are requested
  - **CN**: 将代码或数据的生成延迟到符号真正被请求时

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/IRPartitionLayer.h`, `llvm/ExecutionEngine/Orc/ExecutionUtils.h`, `llvm/ExecutionEngine/Orc/IndirectionUtils.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
