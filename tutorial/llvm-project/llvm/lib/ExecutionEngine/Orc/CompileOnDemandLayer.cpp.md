# CompileOnDemandLayer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/CompileOnDemandLayer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Lazily emit IR on first call.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===----- CompileOnDemandLayer.cpp - Lazily emit IR on first call --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-12
```cpp

#include "llvm/ExecutionEngine/Orc/CompileOnDemandLayer.h"
#include "llvm/ExecutionEngine/Orc/Layer.h"
#include "llvm/IR/Module.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/CompileOnDemandLayer.h`, `llvm/ExecutionEngine/Orc/Layer.h`, `llvm/IR/Module.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/CompileOnDemandLayer.h`, `llvm/ExecutionEngine/Orc/Layer.h`, `llvm/IR/Module.h`。

### Lines 13-22
```cpp
using namespace llvm;
using namespace llvm::orc;

CompileOnDemandLayer::CompileOnDemandLayer(
    ExecutionSession &ES, IRLayer &BaseLayer, LazyCallThroughManager &LCTMgr,
    IndirectStubsManagerBuilder BuildIndirectStubsManager)
    : IRLayer(ES, BaseLayer.getManglingOptions()), BaseLayer(BaseLayer),
      LCTMgr(LCTMgr),
      BuildIndirectStubsManager(std::move(BuildIndirectStubsManager)) {}

```
- **EN**: Introduces declarations for `llvm`, `llvm::orc`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `llvm::orc` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 23-30
```cpp
void CompileOnDemandLayer::setImplMap(ImplSymbolMap *Imp) {
  this->AliaseeImpls = Imp;
}

void CompileOnDemandLayer::emit(
    std::unique_ptr<MaterializationResponsibility> R, ThreadSafeModule TSM) {
  assert(TSM && "Null module");

```
- **EN**: Implements logic around `setImplMap`, `emit`, `assert`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `setImplMap`, `emit`, `assert` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 31-36
```cpp
  auto &ES = getExecutionSession();

  // Sort the callables and non-callables, build re-exports and lodge the
  // actual module with the implementation dylib.
  auto &PDR = getPerDylibResources(R->getTargetJITDylib());

```
- **EN**: Implements logic around `getExecutionSession`, `getPerDylibResources`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `getExecutionSession`, `getPerDylibResources` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 37-46
```cpp
  SymbolAliasMap NonCallables;
  SymbolAliasMap Callables;

  for (auto &KV : R->getSymbols()) {
    auto &Name = KV.first;
    auto &Flags = KV.second;
    if (Flags.isCallable())
      Callables[Name] = SymbolAliasMapEntry(Name, Flags);
    else
      NonCallables[Name] = SymbolAliasMapEntry(Name, Flags);
```
- **EN**: Implements logic around `SymbolAliasMapEntry`.
- **CN**: 围绕 `SymbolAliasMapEntry` 实现具体逻辑。

### Lines 47-56
```cpp
  }

  // Lodge symbols with the implementation dylib.
  if (auto Err = PDR.getImplDylib().define(
          std::make_unique<BasicIRLayerMaterializationUnit>(
              BaseLayer, *getManglingOptions(), std::move(TSM)))) {
    ES.reportError(std::move(Err));
    R->failMaterialization();
    return;
  }
```
- **EN**: Implements logic around `make_unique<BasicIRLayerMaterializationUnit>`, `getManglingOptions`, `reportError`, `failMaterialization`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `make_unique<BasicIRLayerMaterializationUnit>`, `getManglingOptions`, `reportError`, `failMaterialization` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 57-66
```cpp

  if (!NonCallables.empty())
    if (auto Err =
            R->replace(reexports(PDR.getImplDylib(), std::move(NonCallables),
                                 JITDylibLookupFlags::MatchAllSymbols))) {
      getExecutionSession().reportError(std::move(Err));
      R->failMaterialization();
      return;
    }
  if (!Callables.empty()) {
```
- **EN**: Implements logic around `replace`, `getExecutionSession`, `failMaterialization`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `replace`, `getExecutionSession`, `failMaterialization` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 67-76
```cpp
    if (auto Err = R->replace(
            lazyReexports(LCTMgr, PDR.getISManager(), PDR.getImplDylib(),
                          std::move(Callables), AliaseeImpls))) {
      getExecutionSession().reportError(std::move(Err));
      R->failMaterialization();
      return;
    }
  }
}

```
- **EN**: Implements logic around `lazyReexports`, `move`, `getExecutionSession`, `failMaterialization`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `lazyReexports`, `move`, `getExecutionSession`, `failMaterialization` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 77-86
```cpp
CompileOnDemandLayer::PerDylibResources &
CompileOnDemandLayer::getPerDylibResources(JITDylib &TargetD) {
  std::lock_guard<std::mutex> Lock(CODLayerMutex);

  auto I = DylibResources.find(&TargetD);
  if (I == DylibResources.end()) {
    auto &ImplD =
        getExecutionSession().createBareJITDylib(TargetD.getName() + ".impl");
    JITDylibSearchOrder NewLinkOrder;
    TargetD.withLinkOrderDo([&](const JITDylibSearchOrder &TargetLinkOrder) {
```
- **EN**: Implements logic around `getPerDylibResources`, `Lock`, `find`, `getExecutionSession`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `getPerDylibResources`, `Lock`, `find`, `getExecutionSession`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 87-96
```cpp
      NewLinkOrder = TargetLinkOrder;
    });

    assert(!NewLinkOrder.empty() && NewLinkOrder.front().first == &TargetD &&
           NewLinkOrder.front().second ==
               JITDylibLookupFlags::MatchAllSymbols &&
           "TargetD must be at the front of its own search order and match "
           "non-exported symbol");
    NewLinkOrder.insert(std::next(NewLinkOrder.begin()),
                        {&ImplD, JITDylibLookupFlags::MatchAllSymbols});
```
- **EN**: Implements logic around `assert`, `front`, `insert`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `assert`, `front`, `insert` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 97-103
```cpp
    ImplD.setLinkOrder(NewLinkOrder, false);
    TargetD.setLinkOrder(std::move(NewLinkOrder), false);

    PerDylibResources PDR(ImplD, BuildIndirectStubsManager());
    I = DylibResources.insert(std::make_pair(&TargetD, std::move(PDR))).first;
  }

```
- **EN**: Implements logic around `setLinkOrder`, `PDR`, `insert`.
- **CN**: 围绕 `setLinkOrder`, `PDR`, `insert` 实现具体逻辑。

### Lines 104-105
```cpp
  return I->second;
}
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **ORC JIT / ORC JIT**:
  - **EN**: Coordinates JITDylibs, symbol materialization, execution sessions, and asynchronous compilation flows
  - **CN**: 协调 JITDylib、符号物化、ExecutionSession 与异步编译流程
- **Symbol materialization / 符号物化**:
  - **EN**: Defers code or data generation until symbols are requested
  - **CN**: 将代码或数据的生成延迟到符号真正被请求时

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/CompileOnDemandLayer.h`, `llvm/ExecutionEngine/Orc/Layer.h`, `llvm/IR/Module.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, IR
