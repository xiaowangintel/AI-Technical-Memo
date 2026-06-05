# Core.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/Core.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Core ORC APIs (MaterializationUnit, JITDylib, etc.).
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-23
```cpp
//===--- Core.cpp - Core ORC APIs (MaterializationUnit, JITDylib, etc.) ---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ExecutionEngine/Orc/Core.h"

#include "llvm/ADT/STLExtras.h"
#include "llvm/Config/llvm-config.h"
#include "llvm/ExecutionEngine/Orc/AbsoluteSymbols.h"
#include "llvm/ExecutionEngine/Orc/DebugUtils.h"
#include "llvm/ExecutionEngine/Orc/Shared/OrcError.h"
#include "llvm/Support/FormatVariadic.h"
#include "llvm/Support/MSVCErrorWorkarounds.h"
#include "llvm/Support/raw_ostream.h"

#include <condition_variable>
#include <future>
#include <optional>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/Core.h`, `llvm/ADT/STLExtras.h`, `llvm/Config/llvm-config.h`, `llvm/ExecutionEngine/Orc/AbsoluteSymbols.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/Core.h`, `llvm/ADT/STLExtras.h`, `llvm/Config/llvm-config.h`, `llvm/ExecutionEngine/Orc/AbsoluteSymbols.h`。

### Lines 24-44
```cpp
#define DEBUG_TYPE "orc"

namespace llvm {
namespace orc {

char ResourceTrackerDefunct::ID = 0;
char JITDylibDefunct::ID = 0;
char FailedToMaterialize::ID = 0;
char SymbolsNotFound::ID = 0;
char SymbolsCouldNotBeRemoved::ID = 0;
char MissingSymbolDefinitions::ID = 0;
char UnexpectedSymbolDefinitions::ID = 0;
char UnsatisfiedSymbolDependencies::ID = 0;
char MaterializationTask::ID = 0;
char LookupTask::ID = 0;

RegisterDependenciesFunction NoDependenciesToRegister =
    RegisterDependenciesFunction();

void MaterializationUnit::anchor() {}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 45-64
```cpp
ResourceTracker::ResourceTracker(JITDylibSP JD) {
  assert((reinterpret_cast<uintptr_t>(JD.get()) & 0x1) == 0 &&
         "JITDylib must be two byte aligned");
  JD->Retain();
  JDAndFlag.store(reinterpret_cast<uintptr_t>(JD.get()));
}

ResourceTracker::~ResourceTracker() {
  getJITDylib().getExecutionSession().destroyResourceTracker(*this);
  getJITDylib().Release();
}

Error ResourceTracker::remove() {
  return getJITDylib().getExecutionSession().removeResourceTracker(*this);
}

void ResourceTracker::transferTo(ResourceTracker &DstRT) {
  getJITDylib().getExecutionSession().transferResourceTracker(DstRT, *this);
}

```
- **EN**: Implements logic around `ResourceTracker`, `assert`, `Retain`, `store`, and 4 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `ResourceTracker`, `assert`, `Retain`, `store`, and 4 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 65-87
```cpp
void ResourceTracker::makeDefunct() {
  uintptr_t Val = JDAndFlag.load();
  Val |= 0x1U;
  JDAndFlag.store(Val);
}

ResourceManager::~ResourceManager() = default;

ResourceTrackerDefunct::ResourceTrackerDefunct(ResourceTrackerSP RT)
    : RT(std::move(RT)) {}

std::error_code ResourceTrackerDefunct::convertToErrorCode() const {
  return orcError(OrcErrorCode::UnknownORCError);
}

void ResourceTrackerDefunct::log(raw_ostream &OS) const {
  OS << "Resource tracker " << (void *)RT.get() << " became defunct";
}

std::error_code JITDylibDefunct::convertToErrorCode() const {
  return orcError(OrcErrorCode::UnknownORCError);
}

```
- **EN**: Implements logic around `makeDefunct`, `load`, `store`, `~ResourceManager`, and 6 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `makeDefunct`, `load`, `store`, `~ResourceManager`, and 6 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 88-110
```cpp
void JITDylibDefunct::log(raw_ostream &OS) const {
  OS << "JITDylib " << JD->getName() << " (" << (void *)JD.get()
     << ") is defunct";
}

FailedToMaterialize::FailedToMaterialize(
    std::shared_ptr<SymbolStringPool> SSP,
    std::shared_ptr<SymbolDependenceMap> Symbols)
    : SSP(std::move(SSP)), Symbols(std::move(Symbols)) {
  assert(this->SSP && "String pool cannot be null");
  assert(!this->Symbols->empty() && "Can not fail to resolve an empty set");

  // FIXME: Use a new dep-map type for FailedToMaterialize errors so that we
  // don't have to manually retain/release.
  for (auto &[JD, Syms] : *this->Symbols)
    JD->Retain();
}

FailedToMaterialize::~FailedToMaterialize() {
  for (auto &[JD, Syms] : *Symbols)
    JD->Release();
}

```
- **EN**: Implements logic around `log`, `getName`, `FailedToMaterialize`, `SSP`, and 4 more symbols; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `log`, `getName`, `FailedToMaterialize`, `SSP`, and 4 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作。

### Lines 111-130
```cpp
std::error_code FailedToMaterialize::convertToErrorCode() const {
  return orcError(OrcErrorCode::UnknownORCError);
}

void FailedToMaterialize::log(raw_ostream &OS) const {
  OS << "Failed to materialize symbols: " << *Symbols;
}

UnsatisfiedSymbolDependencies::UnsatisfiedSymbolDependencies(
    std::shared_ptr<SymbolStringPool> SSP, JITDylibSP JD,
    SymbolNameSet FailedSymbols, SymbolDependenceMap BadDeps,
    std::string Explanation)
    : SSP(std::move(SSP)), JD(std::move(JD)),
      FailedSymbols(std::move(FailedSymbols)), BadDeps(std::move(BadDeps)),
      Explanation(std::move(Explanation)) {}

std::error_code UnsatisfiedSymbolDependencies::convertToErrorCode() const {
  return orcError(OrcErrorCode::UnknownORCError);
}

```
- **EN**: Implements logic around `convertToErrorCode`, `orcError`, `log`, `UnsatisfiedSymbolDependencies`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `convertToErrorCode`, `orcError`, `log`, `UnsatisfiedSymbolDependencies`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 131-150
```cpp
void UnsatisfiedSymbolDependencies::log(raw_ostream &OS) const {
  OS << "In " << JD->getName() << ", failed to materialize " << FailedSymbols
     << ", due to unsatisfied dependencies " << BadDeps;
  if (!Explanation.empty())
    OS << " (" << Explanation << ")";
}

SymbolsNotFound::SymbolsNotFound(std::shared_ptr<SymbolStringPool> SSP,
                                 SymbolNameSet Symbols)
    : SSP(std::move(SSP)) {
  llvm::append_range(this->Symbols, Symbols);
  assert(!this->Symbols.empty() && "Can not fail to resolve an empty set");
}

SymbolsNotFound::SymbolsNotFound(std::shared_ptr<SymbolStringPool> SSP,
                                 SymbolNameVector Symbols)
    : SSP(std::move(SSP)), Symbols(std::move(Symbols)) {
  assert(!this->Symbols.empty() && "Can not fail to resolve an empty set");
}

```
- **EN**: Implements logic around `log`, `getName`, `SymbolsNotFound`, `SSP`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `log`, `getName`, `SymbolsNotFound`, `SSP`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 151-172
```cpp
std::error_code SymbolsNotFound::convertToErrorCode() const {
  return orcError(OrcErrorCode::UnknownORCError);
}

void SymbolsNotFound::log(raw_ostream &OS) const {
  OS << "Symbols not found: " << Symbols;
}

SymbolsCouldNotBeRemoved::SymbolsCouldNotBeRemoved(
    std::shared_ptr<SymbolStringPool> SSP, SymbolNameSet Symbols)
    : SSP(std::move(SSP)), Symbols(std::move(Symbols)) {
  assert(!this->Symbols.empty() && "Can not fail to resolve an empty set");
}

std::error_code SymbolsCouldNotBeRemoved::convertToErrorCode() const {
  return orcError(OrcErrorCode::UnknownORCError);
}

void SymbolsCouldNotBeRemoved::log(raw_ostream &OS) const {
  OS << "Symbols could not be removed: " << Symbols;
}

```
- **EN**: Implements logic around `convertToErrorCode`, `orcError`, `log`, `SymbolsCouldNotBeRemoved`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `convertToErrorCode`, `orcError`, `log`, `SymbolsCouldNotBeRemoved`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 173-211
```cpp
std::error_code MissingSymbolDefinitions::convertToErrorCode() const {
  return orcError(OrcErrorCode::MissingSymbolDefinitions);
}

void MissingSymbolDefinitions::log(raw_ostream &OS) const {
  OS << "Missing definitions in module " << ModuleName
     << ": " << Symbols;
}

std::error_code UnexpectedSymbolDefinitions::convertToErrorCode() const {
  return orcError(OrcErrorCode::UnexpectedSymbolDefinitions);
}

void UnexpectedSymbolDefinitions::log(raw_ostream &OS) const {
  OS << "Unexpected definitions in module " << ModuleName
     << ": " << Symbols;
}

void SymbolInstance::lookupAsync(LookupAsyncOnCompleteFn OnComplete) const {
  JD->getExecutionSession().lookup(
      LookupKind::Static, {{JD.get(), JITDylibLookupFlags::MatchAllSymbols}},
      SymbolLookupSet(Name), SymbolState::Ready,
      [OnComplete = std::move(OnComplete)
#ifndef NDEBUG
           ,
       Name = this->Name // Captured for the assert below only.
#endif                   // NDEBUG
  ](Expected<SymbolMap> Result) mutable {
        if (Result) {
          assert(Result->size() == 1 && "Unexpected number of results");
          assert(Result->count(Name) &&
                 "Result does not contain expected symbol");
          OnComplete(Result->begin()->second);
        } else
          OnComplete(Result.takeError());
      },
      NoDependenciesToRegister);
}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 212-233
```cpp
AsynchronousSymbolQuery::AsynchronousSymbolQuery(
    const SymbolLookupSet &Symbols, SymbolState RequiredState,
    SymbolsResolvedCallback NotifyComplete)
    : NotifyComplete(std::move(NotifyComplete)), RequiredState(RequiredState) {
  assert(RequiredState >= SymbolState::Resolved &&
         "Cannot query for a symbols that have not reached the resolve state "
         "yet");

  OutstandingSymbolsCount = Symbols.size();

  for (auto &[Name, Flags] : Symbols)
    ResolvedSymbols[Name] = ExecutorSymbolDef();
}

void AsynchronousSymbolQuery::notifySymbolMetRequiredState(
    const SymbolStringPtr &Name, ExecutorSymbolDef Sym) {
  auto I = ResolvedSymbols.find(Name);
  assert(I != ResolvedSymbols.end() &&
         "Resolving symbol outside the requested set");
  assert(I->second == ExecutorSymbolDef() &&
         "Redundantly resolving symbol Name");

```
- **EN**: Implements logic around `AsynchronousSymbolQuery`, `NotifyComplete`, `assert`, `size`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `AsynchronousSymbolQuery`, `NotifyComplete`, `assert`, `size`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作。

### Lines 234-257
```cpp
  // If this is a materialization-side-effects-only symbol then drop it,
  // otherwise update its map entry with its resolved address.
  if (Sym.getFlags().hasMaterializationSideEffectsOnly())
    ResolvedSymbols.erase(I);
  else
    I->second = std::move(Sym);
  --OutstandingSymbolsCount;
}

void AsynchronousSymbolQuery::handleComplete(ExecutionSession &ES) {
  assert(OutstandingSymbolsCount == 0 &&
         "Symbols remain, handleComplete called prematurely");

  class RunQueryCompleteTask : public Task {
  public:
    RunQueryCompleteTask(SymbolMap ResolvedSymbols,
                         SymbolsResolvedCallback NotifyComplete)
        : ResolvedSymbols(std::move(ResolvedSymbols)),
          NotifyComplete(std::move(NotifyComplete)) {}
    void printDescription(raw_ostream &OS) override {
      OS << "Execute query complete callback for " << ResolvedSymbols;
    }
    void run() override { NotifyComplete(std::move(ResolvedSymbols)); }

```
- **EN**: Introduces declarations for `RunQueryCompleteTask`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `RunQueryCompleteTask` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 258-283
```cpp
  private:
    SymbolMap ResolvedSymbols;
    SymbolsResolvedCallback NotifyComplete;
  };

  auto T = std::make_unique<RunQueryCompleteTask>(std::move(ResolvedSymbols),
                                                  std::move(NotifyComplete));
  NotifyComplete = SymbolsResolvedCallback();
  ES.dispatchTask(std::move(T));
}

void AsynchronousSymbolQuery::handleFailed(Error Err) {
  assert(QueryRegistrations.empty() && ResolvedSymbols.empty() &&
         OutstandingSymbolsCount == 0 &&
         "Query should already have been abandoned");
  NotifyComplete(std::move(Err));
  NotifyComplete = SymbolsResolvedCallback();
}

void AsynchronousSymbolQuery::addQueryDependence(JITDylib &JD,
                                                 SymbolStringPtr Name) {
  bool Added = QueryRegistrations[&JD].insert(std::move(Name)).second;
  (void)Added;
  assert(Added && "Duplicate dependence notification?");
}

```
- **EN**: Implements logic around `make_unique<RunQueryCompleteTask>`, `move`, `SymbolsResolvedCallback`, `dispatchTask`, and 5 more symbols; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `make_unique<RunQueryCompleteTask>`, `move`, `SymbolsResolvedCallback`, `dispatchTask`, and 5 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作。

### Lines 284-310
```cpp
void AsynchronousSymbolQuery::removeQueryDependence(
    JITDylib &JD, const SymbolStringPtr &Name) {
  auto QRI = QueryRegistrations.find(&JD);
  assert(QRI != QueryRegistrations.end() &&
         "No dependencies registered for JD");
  assert(QRI->second.count(Name) && "No dependency on Name in JD");
  QRI->second.erase(Name);
  if (QRI->second.empty())
    QueryRegistrations.erase(QRI);
}

void AsynchronousSymbolQuery::dropSymbol(const SymbolStringPtr &Name) {
  auto I = ResolvedSymbols.find(Name);
  assert(I != ResolvedSymbols.end() &&
         "Redundant removal of weakly-referenced symbol");
  ResolvedSymbols.erase(I);
  --OutstandingSymbolsCount;
}

void AsynchronousSymbolQuery::detach() {
  ResolvedSymbols.clear();
  OutstandingSymbolsCount = 0;
  for (auto &[JD, Syms] : QueryRegistrations)
    JD->detachQueryHelper(*this, Syms);
  QueryRegistrations.clear();
}

```
- **EN**: Implements logic around `removeQueryDependence`, `find`, `assert`, `erase`, and 4 more symbols; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `removeQueryDependence`, `find`, `assert`, `erase`, and 4 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作。

### Lines 311-333
```cpp
ReExportsMaterializationUnit::ReExportsMaterializationUnit(
    JITDylib *SourceJD, JITDylibLookupFlags SourceJDLookupFlags,
    SymbolAliasMap Aliases)
    : MaterializationUnit(extractFlags(Aliases)), SourceJD(SourceJD),
      SourceJDLookupFlags(SourceJDLookupFlags), Aliases(std::move(Aliases)) {}

StringRef ReExportsMaterializationUnit::getName() const {
  return "<Reexports>";
}

void ReExportsMaterializationUnit::materialize(
    std::unique_ptr<MaterializationResponsibility> R) {

  auto &ES = R->getTargetJITDylib().getExecutionSession();
  JITDylib &TgtJD = R->getTargetJITDylib();
  JITDylib &SrcJD = SourceJD ? *SourceJD : TgtJD;

  // Find the set of requested aliases and aliasees. Return any unrequested
  // aliases back to the JITDylib so as to not prematurely materialize any
  // aliasees.
  auto RequestedSymbols = R->getRequestedSymbols();
  SymbolAliasMap RequestedAliases;

```
- **EN**: Implements logic around `ReExportsMaterializationUnit`, `MaterializationUnit`, `SourceJDLookupFlags`, `getName`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `ReExportsMaterializationUnit`, `MaterializationUnit`, `SourceJDLookupFlags`, `getName`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 334-353
```cpp
  for (auto &Name : RequestedSymbols) {
    auto I = Aliases.find(Name);
    assert(I != Aliases.end() && "Symbol not found in aliases map?");
    RequestedAliases[Name] = std::move(I->second);
    Aliases.erase(I);
  }

  LLVM_DEBUG({
    ES.runSessionLocked([&]() {
      dbgs() << "materializing reexports: target = " << TgtJD.getName()
             << ", source = " << SrcJD.getName() << " " << RequestedAliases
             << "\n";
    });
  });

  if (!Aliases.empty()) {
    auto Err = SourceJD ? R->replace(reexports(*SourceJD, std::move(Aliases),
                                               SourceJDLookupFlags))
                        : R->replace(symbolAliases(std::move(Aliases)));

```
- **EN**: Implements logic around `find`, `assert`, `move`, `erase`, and 4 more symbols.
- **CN**: 围绕 `find`, `assert`, `move`, `erase`, and 4 more symbols 实现具体逻辑。

### Lines 354-374
```cpp
    if (Err) {
      // FIXME: Should this be reported / treated as failure to materialize?
      // Or should this be treated as a sanctioned bailing-out?
      ES.reportError(std::move(Err));
      R->failMaterialization();
      return;
    }
  }

  // The OnResolveInfo struct will hold the aliases and responsibility for each
  // query in the list.
  struct OnResolveInfo {
    OnResolveInfo(std::unique_ptr<MaterializationResponsibility> R,
                  SymbolAliasMap Aliases)
        : R(std::move(R)), Aliases(std::move(Aliases)) {}

    std::unique_ptr<MaterializationResponsibility> R;
    SymbolAliasMap Aliases;
    std::vector<SymbolDependenceGroup> SDGs;
  };

```
- **EN**: Introduces declarations for `will`, `OnResolveInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `will`, `OnResolveInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 375-396
```cpp
  // Build a list of queries to issue. In each round we build a query for the
  // largest set of aliases that we can resolve without encountering a chain of
  // aliases (e.g. Foo -> Bar, Bar -> Baz). Such a chain would deadlock as the
  // query would be waiting on a symbol that it itself had to resolve. Creating
  // a new query for each link in such a chain eliminates the possibility of
  // deadlock. In practice chains are likely to be rare, and this algorithm will
  // usually result in a single query to issue.

  std::vector<std::pair<SymbolLookupSet, std::shared_ptr<OnResolveInfo>>>
      QueryInfos;
  while (!RequestedAliases.empty()) {
    SymbolNameSet ResponsibilitySymbols;
    SymbolLookupSet QuerySymbols;
    SymbolAliasMap QueryAliases;

    // Collect as many aliases as we can without including a chain.
    for (auto &[Alias, AliasInfo] : RequestedAliases) {
      // Chain detected. Skip this symbol for this round.
      if (&SrcJD == &TgtJD && (QueryAliases.count(AliasInfo.Aliasee) ||
                               RequestedAliases.count(AliasInfo.Aliasee)))
        continue;

```
- **EN**: Implements logic around `count`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `count` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 397-417
```cpp
      ResponsibilitySymbols.insert(Alias);
      QuerySymbols.add(AliasInfo.Aliasee,
                       AliasInfo.AliasFlags.hasMaterializationSideEffectsOnly()
                           ? SymbolLookupFlags::WeaklyReferencedSymbol
                           : SymbolLookupFlags::RequiredSymbol);
      QueryAliases[Alias] = std::move(AliasInfo);
    }

    // Remove the aliases collected this round from the RequestedAliases map.
    for (auto &KV : QueryAliases)
      RequestedAliases.erase(KV.first);

    assert(!QuerySymbols.empty() && "Alias cycle detected!");

    auto NewR = R->delegate(ResponsibilitySymbols);
    if (!NewR) {
      ES.reportError(NewR.takeError());
      R->failMaterialization();
      return;
    }

```
- **EN**: Implements logic around `insert`, `add`, `hasMaterializationSideEffectsOnly`, `move`, and 5 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `insert`, `add`, `hasMaterializationSideEffectsOnly`, `move`, and 5 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 418-440
```cpp
    auto QueryInfo = std::make_shared<OnResolveInfo>(std::move(*NewR),
                                                     std::move(QueryAliases));
    QueryInfos.push_back(
        make_pair(std::move(QuerySymbols), std::move(QueryInfo)));
  }

  // Issue the queries.
  while (!QueryInfos.empty()) {
    auto QuerySymbols = std::move(QueryInfos.back().first);
    auto QueryInfo = std::move(QueryInfos.back().second);

    QueryInfos.pop_back();

    auto RegisterDependencies = [QueryInfo,
                                 &SrcJD](const SymbolDependenceMap &Deps) {
      // If there were no materializing symbols, just bail out.
      if (Deps.empty())
        return;

      // Otherwise the only deps should be on SrcJD.
      assert(Deps.size() == 1 && Deps.count(&SrcJD) &&
             "Unexpected dependencies for reexports");

```
- **EN**: Implements logic around `make_shared<OnResolveInfo>`, `move`, `push_back`, `make_pair`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `make_shared<OnResolveInfo>`, `move`, `push_back`, `make_pair`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 441-478
```cpp
      auto &SrcJDDeps = Deps.find(&SrcJD)->second;

      for (auto &[Alias, AliasInfo] : QueryInfo->Aliases)
        if (SrcJDDeps.count(AliasInfo.Aliasee))
          QueryInfo->SDGs.push_back({{Alias}, {{&SrcJD, {AliasInfo.Aliasee}}}});
    };

    auto OnComplete = [QueryInfo](Expected<SymbolMap> Result) {
      auto &ES = QueryInfo->R->getTargetJITDylib().getExecutionSession();
      if (Result) {
        SymbolMap ResolutionMap;
        for (auto &KV : QueryInfo->Aliases) {
          assert((KV.second.AliasFlags.hasMaterializationSideEffectsOnly() ||
                  Result->count(KV.second.Aliasee)) &&
                 "Result map missing entry?");
          // Don't try to resolve materialization-side-effects-only symbols.
          if (KV.second.AliasFlags.hasMaterializationSideEffectsOnly())
            continue;

          ResolutionMap[KV.first] = {(*Result)[KV.second.Aliasee].getAddress(),
                                     KV.second.AliasFlags};
        }
        if (auto Err = QueryInfo->R->notifyResolved(ResolutionMap)) {
          ES.reportError(std::move(Err));
          QueryInfo->R->failMaterialization();
          return;
        }
        if (auto Err = QueryInfo->R->notifyEmitted(QueryInfo->SDGs)) {
          ES.reportError(std::move(Err));
          QueryInfo->R->failMaterialization();
          return;
        }
      } else {
        ES.reportError(Result.takeError());
        QueryInfo->R->failMaterialization();
      }
    };

```
- **EN**: Implements logic around `find`, `push_back`, `getTargetJITDylib`, `assert`, and 4 more symbols; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `find`, `push_back`, `getTargetJITDylib`, `assert`, and 4 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作。

### Lines 479-498
```cpp
    ES.lookup(LookupKind::Static,
              JITDylibSearchOrder({{&SrcJD, SourceJDLookupFlags}}),
              QuerySymbols, SymbolState::Resolved, std::move(OnComplete),
              std::move(RegisterDependencies));
  }
}

void ReExportsMaterializationUnit::discard(const JITDylib &JD,
                                           const SymbolStringPtr &Name) {
  assert(Aliases.count(Name) &&
         "Symbol not covered by this MaterializationUnit");
  Aliases.erase(Name);
}

MaterializationUnit::Interface
ReExportsMaterializationUnit::extractFlags(const SymbolAliasMap &Aliases) {
  SymbolFlagsMap SymbolFlags;
  for (auto &KV : Aliases)
    SymbolFlags[KV.first] = KV.second.AliasFlags;

```
- **EN**: Implements logic around `lookup`, `JITDylibSearchOrder`, `move`, `discard`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `lookup`, `JITDylibSearchOrder`, `move`, `discard`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作。

### Lines 499-520
```cpp
  return MaterializationUnit::Interface(std::move(SymbolFlags), nullptr);
}

Expected<SymbolAliasMap> buildSimpleReexportsAliasMap(JITDylib &SourceJD,
                                                      SymbolNameSet Symbols) {
  SymbolLookupSet LookupSet(Symbols);
  auto Flags = SourceJD.getExecutionSession().lookupFlags(
      LookupKind::Static, {{&SourceJD, JITDylibLookupFlags::MatchAllSymbols}},
      SymbolLookupSet(std::move(Symbols)));

  if (!Flags)
    return Flags.takeError();

  SymbolAliasMap Result;
  for (auto &Name : Symbols) {
    assert(Flags->count(Name) && "Missing entry in flags map");
    Result[Name] = SymbolAliasMapEntry(Name, (*Flags)[Name]);
  }

  return Result;
}

```
- **EN**: Implements logic around `Interface`, `buildSimpleReexportsAliasMap`, `LookupSet`, `getExecutionSession`, and 4 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `Interface`, `buildSimpleReexportsAliasMap`, `LookupSet`, `getExecutionSession`, and 4 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 521-540
```cpp
class InProgressLookupState {
public:
  // FIXME: Reduce the number of SymbolStringPtrs here. See
  //        https://github.com/llvm/llvm-project/issues/55576.

  InProgressLookupState(LookupKind K, JITDylibSearchOrder SearchOrder,
                        SymbolLookupSet LookupSet, SymbolState RequiredState)
      : K(K), SearchOrder(std::move(SearchOrder)),
        LookupSet(std::move(LookupSet)), RequiredState(RequiredState) {
    DefGeneratorCandidates = this->LookupSet;
  }
  virtual ~InProgressLookupState() = default;
  virtual void complete(std::unique_ptr<InProgressLookupState> IPLS) = 0;
  virtual void fail(Error Err) = 0;

  LookupKind K;
  JITDylibSearchOrder SearchOrder;
  SymbolLookupSet LookupSet;
  SymbolState RequiredState;

```
- **EN**: Introduces declarations for `InProgressLookupState`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `InProgressLookupState` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 541-562
```cpp
  size_t CurSearchOrderIndex = 0;
  bool NewJITDylib = true;
  SymbolLookupSet DefGeneratorCandidates;
  SymbolLookupSet DefGeneratorNonCandidates;

  enum {
    NotInGenerator,      // Not currently using a generator.
    ResumedForGenerator, // Resumed after being auto-suspended before generator.
    InGenerator          // Currently using generator.
  } GenState = NotInGenerator;
  std::vector<std::weak_ptr<DefinitionGenerator>> CurDefGeneratorStack;
};

class InProgressLookupFlagsState : public InProgressLookupState {
public:
  InProgressLookupFlagsState(
      LookupKind K, JITDylibSearchOrder SearchOrder, SymbolLookupSet LookupSet,
      unique_function<void(Expected<SymbolFlagsMap>)> OnComplete)
      : InProgressLookupState(K, std::move(SearchOrder), std::move(LookupSet),
                              SymbolState::NeverSearched),
        OnComplete(std::move(OnComplete)) {}

```
- **EN**: Introduces declarations for `InProgressLookupFlagsState`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `InProgressLookupFlagsState` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 563-585
```cpp
  void complete(std::unique_ptr<InProgressLookupState> IPLS) override {
    auto &ES = SearchOrder.front().first->getExecutionSession();
    ES.OL_completeLookupFlags(std::move(IPLS), std::move(OnComplete));
  }

  void fail(Error Err) override { OnComplete(std::move(Err)); }

private:
  unique_function<void(Expected<SymbolFlagsMap>)> OnComplete;
};

class InProgressFullLookupState : public InProgressLookupState {
public:
  InProgressFullLookupState(LookupKind K, JITDylibSearchOrder SearchOrder,
                            SymbolLookupSet LookupSet,
                            SymbolState RequiredState,
                            std::shared_ptr<AsynchronousSymbolQuery> Q,
                            RegisterDependenciesFunction RegisterDependencies)
      : InProgressLookupState(K, std::move(SearchOrder), std::move(LookupSet),
                              RequiredState),
        Q(std::move(Q)), RegisterDependencies(std::move(RegisterDependencies)) {
  }

```
- **EN**: Introduces declarations for `InProgressFullLookupState`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `InProgressFullLookupState` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 586-607
```cpp
  void complete(std::unique_ptr<InProgressLookupState> IPLS) override {
    auto &ES = SearchOrder.front().first->getExecutionSession();
    ES.OL_completeLookup(std::move(IPLS), std::move(Q),
                         std::move(RegisterDependencies));
  }

  void fail(Error Err) override {
    Q->detach();
    Q->handleFailed(std::move(Err));
  }

private:
  std::shared_ptr<AsynchronousSymbolQuery> Q;
  RegisterDependenciesFunction RegisterDependencies;
};

ReexportsGenerator::ReexportsGenerator(JITDylib &SourceJD,
                                       JITDylibLookupFlags SourceJDLookupFlags,
                                       SymbolPredicate Allow)
    : SourceJD(SourceJD), SourceJDLookupFlags(SourceJDLookupFlags),
      Allow(std::move(Allow)) {}

```
- **EN**: Implements logic around `complete`, `front`, `OL_completeLookup`, `move`, and 6 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `complete`, `front`, `OL_completeLookup`, `move`, and 6 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 608-628
```cpp
Error ReexportsGenerator::tryToGenerate(LookupState &LS, LookupKind K,
                                        JITDylib &JD,
                                        JITDylibLookupFlags JDLookupFlags,
                                        const SymbolLookupSet &LookupSet) {
  assert(&JD != &SourceJD && "Cannot re-export from the same dylib");

  // Use lookupFlags to find the subset of symbols that match our lookup.
  auto Flags = JD.getExecutionSession().lookupFlags(
      K, {{&SourceJD, JDLookupFlags}}, LookupSet);
  if (!Flags)
    return Flags.takeError();

  // Create an alias map.
  orc::SymbolAliasMap AliasMap;
  for (auto &KV : *Flags)
    if (!Allow || Allow(KV.first))
      AliasMap[KV.first] = SymbolAliasMapEntry(KV.first, KV.second);

  if (AliasMap.empty())
    return Error::success();

```
- **EN**: Implements logic around `tryToGenerate`, `assert`, `getExecutionSession`, `takeError`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `tryToGenerate`, `assert`, `getExecutionSession`, `takeError`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 629-648
```cpp
  // Define the re-exports.
  return JD.define(reexports(SourceJD, AliasMap, SourceJDLookupFlags));
}

LookupState::LookupState(std::unique_ptr<InProgressLookupState> IPLS)
    : IPLS(std::move(IPLS)) {}

void LookupState::reset(InProgressLookupState *IPLS) { this->IPLS.reset(IPLS); }

LookupState::LookupState() = default;
LookupState::LookupState(LookupState &&) = default;
LookupState &LookupState::operator=(LookupState &&) = default;
LookupState::~LookupState() = default;

void LookupState::continueLookup(Error Err) {
  assert(IPLS && "Cannot call continueLookup on empty LookupState");
  auto &ES = IPLS->SearchOrder.begin()->first->getExecutionSession();
  ES.OL_applyQueryPhase1(std::move(IPLS), std::move(Err));
}

```
- **EN**: Implements logic around `define`, `LookupState`, `IPLS`, `reset`, and 5 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `define`, `LookupState`, `IPLS`, `reset`, and 5 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 649-675
```cpp
DefinitionGenerator::~DefinitionGenerator() {
  std::deque<LookupState> LookupsToFail;
  {
    std::lock_guard<std::mutex> Lock(M);
    std::swap(PendingLookups, LookupsToFail);
    InUse = false;
  }

  for (auto &LS : LookupsToFail)
    LS.continueLookup(make_error<StringError>(
        "Query waiting on DefinitionGenerator that was destroyed",
        inconvertibleErrorCode()));
}

JITDylib::~JITDylib() {
  LLVM_DEBUG(dbgs() << "Destroying JITDylib " << getName() << "\n");
}

Error JITDylib::clear() {
  std::vector<ResourceTrackerSP> TrackersToRemove;
  ES.runSessionLocked([&]() {
    assert(State != Closed && "JD is defunct");
    for (auto &KV : TrackerSymbols)
      TrackersToRemove.push_back(KV.first);
    TrackersToRemove.push_back(getDefaultResourceTracker());
  });

```
- **EN**: Implements logic around `~DefinitionGenerator`, `Lock`, `swap`, `continueLookup`, and 6 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `~DefinitionGenerator`, `Lock`, `swap`, `continueLookup`, and 6 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 676-698
```cpp
  Error Err = Error::success();
  for (auto &RT : TrackersToRemove)
    Err = joinErrors(std::move(Err), RT->remove());
  return Err;
}

ResourceTrackerSP JITDylib::getDefaultResourceTracker() {
  return ES.runSessionLocked([this] {
    assert(State != Closed && "JD is defunct");
    if (!DefaultTracker)
      DefaultTracker = new ResourceTracker(this);
    return DefaultTracker;
  });
}

ResourceTrackerSP JITDylib::createResourceTracker() {
  return ES.runSessionLocked([this] {
    assert(State == Open && "JD is defunct");
    ResourceTrackerSP RT = new ResourceTracker(this);
    return RT;
  });
}

```
- **EN**: Implements logic around `success`, `joinErrors`, `getDefaultResourceTracker`, `runSessionLocked`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `joinErrors`, `getDefaultResourceTracker`, `runSessionLocked`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 699-719
```cpp
void JITDylib::removeGenerator(DefinitionGenerator &G) {
  // DefGenerator moved into TmpDG to ensure that it's destroyed outside the
  // session lock (since it may have to send errors to pending queries).
  std::shared_ptr<DefinitionGenerator> TmpDG;

  ES.runSessionLocked([&] {
    assert(State == Open && "JD is defunct");
    auto I = llvm::find_if(DefGenerators,
                           [&](const std::shared_ptr<DefinitionGenerator> &H) {
                             return H.get() == &G;
                           });
    assert(I != DefGenerators.end() && "Generator not found");
    TmpDG = std::move(*I);
    DefGenerators.erase(I);
  });
}

Expected<SymbolFlagsMap>
JITDylib::defineMaterializing(MaterializationResponsibility &FromMR,
                              SymbolFlagsMap SymbolFlags) {

```
- **EN**: Implements logic around `removeGenerator`, `runSessionLocked`, `assert`, `find_if`, and 4 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `removeGenerator`, `runSessionLocked`, `assert`, `find_if`, and 4 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 720-743
```cpp
  return ES.runSessionLocked([&]() -> Expected<SymbolFlagsMap> {
    if (FromMR.RT->isDefunct())
      return make_error<ResourceTrackerDefunct>(FromMR.RT);

    std::vector<NonOwningSymbolStringPtr> AddedSyms;
    std::vector<NonOwningSymbolStringPtr> RejectedWeakDefs;

    for (auto &[Name, Flags] : SymbolFlags) {
      auto EntryItr = Symbols.find(Name);

      // If the entry already exists...
      if (EntryItr != Symbols.end()) {

        // If this is a strong definition then error out.
        if (!Flags.isWeak()) {
          // Remove any symbols already added.
          for (auto &S : AddedSyms)
            Symbols.erase(Symbols.find_as(S));

          // FIXME: Return all duplicates.
          return make_error<DuplicateDefinition>(
              std::string(*Name), "defineMaterializing operation");
        }

```
- **EN**: Implements logic around `runSessionLocked`, `make_error<ResourceTrackerDefunct>`, `find`, `erase`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `runSessionLocked`, `make_error<ResourceTrackerDefunct>`, `find`, `erase`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 744-764
```cpp
        // Otherwise just make a note to discard this symbol after the loop.
        RejectedWeakDefs.push_back(NonOwningSymbolStringPtr(Name));
        continue;
      } else
        EntryItr =
          Symbols.insert(std::make_pair(Name, SymbolTableEntry(Flags))).first;

      AddedSyms.push_back(NonOwningSymbolStringPtr(Name));
      EntryItr->second.setState(SymbolState::Materializing);
    }

    // Remove any rejected weak definitions from the SymbolFlags map.
    while (!RejectedWeakDefs.empty()) {
      SymbolFlags.erase(SymbolFlags.find_as(RejectedWeakDefs.back()));
      RejectedWeakDefs.pop_back();
    }

    return SymbolFlags;
  });
}

```
- **EN**: Implements logic around `push_back`, `insert`, `setState`, `erase`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `insert`, `setState`, `erase`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 765-788
```cpp
Error JITDylib::replace(MaterializationResponsibility &FromMR,
                        std::unique_ptr<MaterializationUnit> MU) {
  assert(MU != nullptr && "Can not replace with a null MaterializationUnit");
  std::unique_ptr<MaterializationUnit> MustRunMU;
  std::unique_ptr<MaterializationResponsibility> MustRunMR;

  auto Err =
      ES.runSessionLocked([&, this]() -> Error {
        if (FromMR.RT->isDefunct())
          return make_error<ResourceTrackerDefunct>(std::move(FromMR.RT));

#ifndef NDEBUG
        for (auto &KV : MU->getSymbols()) {
          auto SymI = Symbols.find(KV.first);
          assert(SymI != Symbols.end() && "Replacing unknown symbol");
          assert(SymI->second.getState() == SymbolState::Materializing &&
                 "Can not replace a symbol that ha is not materializing");
          assert(!SymI->second.hasMaterializerAttached() &&
                 "Symbol should not have materializer attached already");
          assert(UnmaterializedInfos.count(KV.first) == 0 &&
                 "Symbol being replaced should have no UnmaterializedInfo");
        }
#endif // NDEBUG

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 789-819
```cpp
        // If the tracker is defunct we need to bail out immediately.

        // If any symbol has pending queries against it then we need to
        // materialize MU immediately.
        for (auto &KV : MU->getSymbols()) {
          auto MII = MaterializingInfos.find(KV.first);
          if (MII != MaterializingInfos.end()) {
            if (MII->second.hasQueriesPending()) {
              MustRunMR = ES.createMaterializationResponsibility(
                  *FromMR.RT, std::move(MU->SymbolFlags),
                  std::move(MU->InitSymbol));
              MustRunMU = std::move(MU);
              return Error::success();
            }
          }
        }

        // Otherwise, make MU responsible for all the symbols.
        auto UMI = std::make_shared<UnmaterializedInfo>(std::move(MU),
                                                        FromMR.RT.get());
        for (auto &KV : UMI->MU->getSymbols()) {
          auto SymI = Symbols.find(KV.first);
          assert(SymI->second.getState() == SymbolState::Materializing &&
                 "Can not replace a symbol that is not materializing");
          assert(!SymI->second.hasMaterializerAttached() &&
                 "Can not replace a symbol that has a materializer attached");
          assert(UnmaterializedInfos.count(KV.first) == 0 &&
                 "Unexpected materializer entry in map");
          SymI->second.setAddress(SymI->second.getAddress());
          SymI->second.setMaterializerAttached(true);

```
- **EN**: Implements logic around `find`, `createMaterializationResponsibility`, `move`, `success`, and 5 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `createMaterializationResponsibility`, `move`, `success`, and 5 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 820-839
```cpp
          auto &UMIEntry = UnmaterializedInfos[KV.first];
          assert((!UMIEntry || !UMIEntry->MU) &&
                 "Replacing symbol with materializer still attached");
          UMIEntry = UMI;
        }

        return Error::success();
      });

  if (Err)
    return Err;

  if (MustRunMU) {
    assert(MustRunMR && "MustRunMU set implies MustRunMR set");
    ES.dispatchTask(std::make_unique<MaterializationTask>(
        std::move(MustRunMU), std::move(MustRunMR)));
  } else {
    assert(!MustRunMR && "MustRunMU unset implies MustRunMR unset");
  }

```
- **EN**: Implements logic around `assert`, `success`, `dispatchTask`, `move`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `success`, `dispatchTask`, `move` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 840-861
```cpp
  return Error::success();
}

Expected<std::unique_ptr<MaterializationResponsibility>>
JITDylib::delegate(MaterializationResponsibility &FromMR,
                   SymbolFlagsMap SymbolFlags, SymbolStringPtr InitSymbol) {

  return ES.runSessionLocked(
      [&]() -> Expected<std::unique_ptr<MaterializationResponsibility>> {
        if (FromMR.RT->isDefunct())
          return make_error<ResourceTrackerDefunct>(std::move(FromMR.RT));

        return ES.createMaterializationResponsibility(
            *FromMR.RT, std::move(SymbolFlags), std::move(InitSymbol));
      });
}

SymbolNameSet
JITDylib::getRequestedSymbols(const SymbolFlagsMap &SymbolFlags) const {
  return ES.runSessionLocked([&]() {
    SymbolNameSet RequestedSymbols;

```
- **EN**: Implements logic around `success`, `delegate`, `runSessionLocked`, `make_error<ResourceTrackerDefunct>`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `delegate`, `runSessionLocked`, `make_error<ResourceTrackerDefunct>`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 862-884
```cpp
    for (auto &KV : SymbolFlags) {
      assert(Symbols.count(KV.first) && "JITDylib does not cover this symbol?");
      assert(Symbols.find(KV.first)->second.getState() !=
                 SymbolState::NeverSearched &&
             Symbols.find(KV.first)->second.getState() != SymbolState::Ready &&
             "getRequestedSymbols can only be called for symbols that have "
             "started materializing");
      auto I = MaterializingInfos.find(KV.first);
      if (I == MaterializingInfos.end())
        continue;

      if (I->second.hasQueriesPending())
        RequestedSymbols.insert(KV.first);
    }

    return RequestedSymbols;
  });
}

Error JITDylib::resolve(MaterializationResponsibility &MR,
                        const SymbolMap &Resolved) {
  AsynchronousSymbolQuerySet CompletedQueries;

```
- **EN**: Implements logic around `assert`, `find`, `insert`, `resolve`; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `find`, `insert`, `resolve` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 885-905
```cpp
  if (auto Err = ES.runSessionLocked([&, this]() -> Error {
        if (MR.RT->isDefunct())
          return make_error<ResourceTrackerDefunct>(MR.RT);

        if (State != Open)
          return make_error<StringError>("JITDylib " + getName() +
                                             " is defunct",
                                         inconvertibleErrorCode());

        struct WorklistEntry {
          SymbolTable::iterator SymI;
          ExecutorSymbolDef ResolvedSym;
        };

        SymbolNameSet SymbolsInErrorState;
        std::vector<WorklistEntry> Worklist;
        Worklist.reserve(Resolved.size());

        // Build worklist and check for any symbols in the error state.
        for (const auto &KV : Resolved) {

```
- **EN**: Introduces declarations for `WorklistEntry`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `WorklistEntry` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 906-930
```cpp
          assert(!KV.second.getFlags().hasError() &&
                 "Resolution result can not have error flag set");

          auto SymI = Symbols.find(KV.first);

          assert(SymI != Symbols.end() && "Symbol not found");
          assert(!SymI->second.hasMaterializerAttached() &&
                 "Resolving symbol with materializer attached?");
          assert(SymI->second.getState() == SymbolState::Materializing &&
                 "Symbol should be materializing");
          assert(SymI->second.getAddress() == ExecutorAddr() &&
                 "Symbol has already been resolved");

          if (SymI->second.getFlags().hasError())
            SymbolsInErrorState.insert(KV.first);
          else {
            if (SymI->second.getFlags() & JITSymbolFlags::Common) {
              [[maybe_unused]] auto WeakOrCommon =
                  JITSymbolFlags::Weak | JITSymbolFlags::Common;
              assert((KV.second.getFlags() & WeakOrCommon) &&
                     "Common symbols must be resolved as common or weak");
              assert((KV.second.getFlags() & ~WeakOrCommon) ==
                         (SymI->second.getFlags() & ~JITSymbolFlags::Common) &&
                     "Resolving symbol with incorrect flags");

```
- **EN**: Implements logic around `assert`, `find`, `insert`, `getFlags`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `assert`, `find`, `insert`, `getFlags` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 931-953
```cpp
            } else
              assert(KV.second.getFlags() == SymI->second.getFlags() &&
                     "Resolved flags should match the declared flags");

            Worklist.push_back(
                {SymI, {KV.second.getAddress(), SymI->second.getFlags()}});
          }
        }

        // If any symbols were in the error state then bail out.
        if (!SymbolsInErrorState.empty()) {
          auto FailedSymbolsDepMap = std::make_shared<SymbolDependenceMap>();
          (*FailedSymbolsDepMap)[this] = std::move(SymbolsInErrorState);
          return make_error<FailedToMaterialize>(
              getExecutionSession().getSymbolStringPool(),
              std::move(FailedSymbolsDepMap));
        }

        while (!Worklist.empty()) {
          auto SymI = Worklist.back().SymI;
          auto ResolvedSym = Worklist.back().ResolvedSym;
          Worklist.pop_back();

```
- **EN**: Implements logic around `assert`, `push_back`, `getAddress`, `make_shared<SymbolDependenceMap>`, and 5 more symbols; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `push_back`, `getAddress`, `make_shared<SymbolDependenceMap>`, and 5 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 954-973
```cpp
          auto &Name = SymI->first;

          // Resolved symbols can not be weak: discard the weak flag.
          JITSymbolFlags ResolvedFlags = ResolvedSym.getFlags();
          SymI->second.setAddress(ResolvedSym.getAddress());
          SymI->second.setFlags(ResolvedFlags);
          SymI->second.setState(SymbolState::Resolved);

          auto MII = MaterializingInfos.find(Name);
          if (MII == MaterializingInfos.end())
            continue;

          auto &MI = MII->second;
          for (auto &Q : MI.takeQueriesMeeting(SymbolState::Resolved)) {
            Q->notifySymbolMetRequiredState(Name, ResolvedSym);
            if (Q->isComplete())
              CompletedQueries.insert(std::move(Q));
          }
        }

```
- **EN**: Implements logic around `getFlags`, `setAddress`, `setFlags`, `setState`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getFlags`, `setAddress`, `setFlags`, `setState`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 974-998
```cpp
        return Error::success();
      }))
    return Err;

  // Otherwise notify all the completed queries.
  for (auto &Q : CompletedQueries) {
    assert(Q->isComplete() && "Q not completed");
    Q->handleComplete(ES);
  }

  return Error::success();
}

void JITDylib::unlinkMaterializationResponsibility(
    MaterializationResponsibility &MR) {
  ES.runSessionLocked([&]() {
    auto I = TrackerMRs.find(MR.RT.get());
    assert(I != TrackerMRs.end() && "No MRs in TrackerMRs list for RT");
    assert(I->second.count(&MR) && "MR not in TrackerMRs list for RT");
    I->second.erase(&MR);
    if (I->second.empty())
      TrackerMRs.erase(MR.RT.get());
  });
}

```
- **EN**: Implements logic around `success`, `assert`, `handleComplete`, `unlinkMaterializationResponsibility`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `assert`, `handleComplete`, `unlinkMaterializationResponsibility`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 999-1024
```cpp
void JITDylib::shrinkMaterializationInfoMemory() {
  // DenseMap::erase never shrinks its storage; use clear to heuristically free
  // memory since we may have long-lived JDs after linking is done.

  if (UnmaterializedInfos.empty())
    UnmaterializedInfos.clear();

  if (MaterializingInfos.empty())
    MaterializingInfos.clear();
}

void JITDylib::setLinkOrder(JITDylibSearchOrder NewLinkOrder,
                            bool LinkAgainstThisJITDylibFirst) {
  ES.runSessionLocked([&]() {
    assert(State == Open && "JD is defunct");
    if (LinkAgainstThisJITDylibFirst) {
      LinkOrder.clear();
      if (NewLinkOrder.empty() || NewLinkOrder.front().first != this)
        LinkOrder.push_back(
            std::make_pair(this, JITDylibLookupFlags::MatchAllSymbols));
      llvm::append_range(LinkOrder, NewLinkOrder);
    } else
      LinkOrder = std::move(NewLinkOrder);
  });
}

```
- **EN**: Implements logic around `shrinkMaterializationInfoMemory`, `clear`, `setLinkOrder`, `runSessionLocked`, and 5 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `shrinkMaterializationInfoMemory`, `clear`, `setLinkOrder`, `runSessionLocked`, and 5 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 1025-1052
```cpp
void JITDylib::addToLinkOrder(const JITDylibSearchOrder &NewLinks) {
  ES.runSessionLocked([&]() {
    for (auto &KV : NewLinks) {
      // Skip elements of NewLinks that are already in the link order.
      if (llvm::is_contained(LinkOrder, KV))
        continue;

      LinkOrder.push_back(std::move(KV));
    }
  });
}

void JITDylib::addToLinkOrder(JITDylib &JD, JITDylibLookupFlags JDLookupFlags) {
  ES.runSessionLocked([&]() { LinkOrder.push_back({&JD, JDLookupFlags}); });
}

void JITDylib::replaceInLinkOrder(JITDylib &OldJD, JITDylib &NewJD,
                                  JITDylibLookupFlags JDLookupFlags) {
  ES.runSessionLocked([&]() {
    assert(State == Open && "JD is defunct");
    for (auto &KV : LinkOrder)
      if (KV.first == &OldJD) {
        KV = {&NewJD, JDLookupFlags};
        break;
      }
  });
}

```
- **EN**: Implements logic around `addToLinkOrder`, `runSessionLocked`, `push_back`, `replaceInLinkOrder`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `addToLinkOrder`, `runSessionLocked`, `push_back`, `replaceInLinkOrder`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 1053-1073
```cpp
void JITDylib::removeFromLinkOrder(JITDylib &JD) {
  ES.runSessionLocked([&]() {
    assert(State == Open && "JD is defunct");
    auto I = llvm::find_if(LinkOrder,
                           [&](const JITDylibSearchOrder::value_type &KV) {
                             return KV.first == &JD;
                           });
    if (I != LinkOrder.end())
      LinkOrder.erase(I);
  });
}

Error JITDylib::remove(const SymbolNameSet &Names) {
  return ES.runSessionLocked([&]() -> Error {
    assert(State == Open && "JD is defunct");
    using SymbolMaterializerItrPair =
        std::pair<SymbolTable::iterator, UnmaterializedInfosMap::iterator>;
    std::vector<SymbolMaterializerItrPair> SymbolsToRemove;
    SymbolNameSet Missing;
    SymbolNameSet Materializing;

```
- **EN**: Implements logic around `removeFromLinkOrder`, `runSessionLocked`, `assert`, `find_if`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `removeFromLinkOrder`, `runSessionLocked`, `assert`, `find_if`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 1074-1095
```cpp
    for (auto &Name : Names) {
      auto I = Symbols.find(Name);

      // Note symbol missing.
      if (I == Symbols.end()) {
        Missing.insert(Name);
        continue;
      }

      // Note symbol materializing.
      if (I->second.getState() != SymbolState::NeverSearched &&
          I->second.getState() != SymbolState::Ready) {
        Materializing.insert(Name);
        continue;
      }

      auto UMII = I->second.hasMaterializerAttached()
                      ? UnmaterializedInfos.find(Name)
                      : UnmaterializedInfos.end();
      SymbolsToRemove.push_back(std::make_pair(I, UMII));
    }

```
- **EN**: Implements logic around `find`, `insert`, `getState`, `hasMaterializerAttached`, and 2 more symbols.
- **CN**: 围绕 `find`, `insert`, `getState`, `hasMaterializerAttached`, and 2 more symbols 实现具体逻辑。

### Lines 1096-1115
```cpp
    // If any of the symbols are not defined, return an error.
    if (!Missing.empty())
      return make_error<SymbolsNotFound>(ES.getSymbolStringPool(),
                                         std::move(Missing));

    // If any of the symbols are currently materializing, return an error.
    if (!Materializing.empty())
      return make_error<SymbolsCouldNotBeRemoved>(ES.getSymbolStringPool(),
                                                  std::move(Materializing));

    // Remove the symbols.
    for (auto &SymbolMaterializerItrPair : SymbolsToRemove) {
      auto UMII = SymbolMaterializerItrPair.second;

      // If there is a materializer attached, call discard.
      if (UMII != UnmaterializedInfos.end()) {
        UMII->second->MU->doDiscard(*this, UMII->first);
        UnmaterializedInfos.erase(UMII);
      }

```
- **EN**: Implements logic around `make_error<SymbolsNotFound>`, `move`, `make_error<SymbolsCouldNotBeRemoved>`, `doDiscard`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<SymbolsNotFound>`, `move`, `make_error<SymbolsCouldNotBeRemoved>`, `doDiscard`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 1116-1147
```cpp
      auto SymI = SymbolMaterializerItrPair.first;
      Symbols.erase(SymI);
    }

    shrinkMaterializationInfoMemory();

    return Error::success();
  });
}

void JITDylib::dump(raw_ostream &OS) {
  ES.runSessionLocked([&, this]() {
    OS << "JITDylib \"" << getName() << "\" (ES: "
       << format("0x%016" PRIx64, reinterpret_cast<uintptr_t>(&ES))
       << ", State = ";
    switch (State) {
    case Open:
      OS << "Open";
      break;
    case Closing:
      OS << "Closing";
      break;
    case Closed:
      OS << "Closed";
      break;
    }
    OS << ")\n";
    if (State == Closed)
      return;
    OS << "Link order: " << LinkOrder << "\n"
       << "Symbol table:\n";

```
- **EN**: Implements logic around `erase`, `shrinkMaterializationInfoMemory`, `success`, `dump`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `erase`, `shrinkMaterializationInfoMemory`, `success`, `dump`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 1148-1173
```cpp
    // Sort symbols so we get a deterministic order and can check them in tests.
    std::vector<std::pair<SymbolStringPtr, SymbolTableEntry *>> SymbolsSorted;
    for (auto &KV : Symbols)
      SymbolsSorted.emplace_back(KV.first, &KV.second);
    std::sort(SymbolsSorted.begin(), SymbolsSorted.end(),
              [](const auto &L, const auto &R) { return *L.first < *R.first; });

    for (auto &KV : SymbolsSorted) {
      OS << "    \"" << *KV.first << "\": ";
      if (auto Addr = KV.second->getAddress())
        OS << Addr;
      else
        OS << "<not resolved> ";

      OS << " " << KV.second->getFlags() << " " << KV.second->getState();

      if (KV.second->hasMaterializerAttached()) {
        OS << " (Materializer ";
        auto I = UnmaterializedInfos.find(KV.first);
        assert(I != UnmaterializedInfos.end() &&
               "Lazy symbol should have UnmaterializedInfo");
        OS << I->second->MU.get() << ", " << I->second->MU->getName() << ")\n";
      } else
        OS << "\n";
    }

```
- **EN**: Implements logic around `emplace_back`, `sort`, `getFlags`, `find`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `emplace_back`, `sort`, `getFlags`, `find`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 1174-1197
```cpp
    if (!MaterializingInfos.empty())
      OS << "  MaterializingInfos entries:\n";
    for (auto &KV : MaterializingInfos) {
      OS << "    \"" << *KV.first << "\":\n"
         << "      " << KV.second.pendingQueries().size()
         << " pending queries: { ";
      for (const auto &Q : KV.second.pendingQueries())
        OS << Q.get() << " (" << Q->getRequiredState() << ") ";
      OS << "}\n";
    }
  });
}

void JITDylib::MaterializingInfo::addQuery(
    std::shared_ptr<AsynchronousSymbolQuery> Q) {

  auto I = llvm::lower_bound(
      llvm::reverse(PendingQueries), Q->getRequiredState(),
      [](const std::shared_ptr<AsynchronousSymbolQuery> &V, SymbolState S) {
        return V->getRequiredState() <= S;
      });
  PendingQueries.insert(I.base(), std::move(Q));
}

```
- **EN**: Implements logic around `pendingQueries`, `get`, `addQuery`, `lower_bound`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `pendingQueries`, `get`, `addQuery`, `lower_bound`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 1198-1219
```cpp
void JITDylib::MaterializingInfo::removeQuery(
    const AsynchronousSymbolQuery &Q) {
  // FIXME: Implement 'find_as' for shared_ptr<T>/T*.
  auto I = llvm::find_if(
      PendingQueries, [&Q](const std::shared_ptr<AsynchronousSymbolQuery> &V) {
        return V.get() == &Q;
      });
  if (I != PendingQueries.end())
    PendingQueries.erase(I);
}

JITDylib::AsynchronousSymbolQueryList
JITDylib::MaterializingInfo::takeQueriesMeeting(SymbolState RequiredState) {
  AsynchronousSymbolQueryList Result;
  while (!PendingQueries.empty()) {
    if (PendingQueries.back()->getRequiredState() > RequiredState)
      break;

    Result.push_back(std::move(PendingQueries.back()));
    PendingQueries.pop_back();
  }

```
- **EN**: Implements logic around `removeQuery`, `find_if`, `get`, `erase`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `removeQuery`, `find_if`, `get`, `erase`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 1220-1239
```cpp
  return Result;
}

JITDylib::JITDylib(ExecutionSession &ES, std::string Name)
    : JITLinkDylib(std::move(Name)), ES(ES) {
  LinkOrder.push_back({this, JITDylibLookupFlags::MatchAllSymbols});
}

JITDylib::RemoveTrackerResult JITDylib::IL_removeTracker(ResourceTracker &RT) {
  // Note: Should be called under the session lock.
  assert(State != Closed && "JD is defunct");

  SymbolNameVector SymbolsToRemove;
  SymbolNameVector SymbolsToFail;

  if (&RT == DefaultTracker.get()) {
    SymbolNameSet TrackedSymbols;
    for (auto &KV : TrackerSymbols)
      TrackedSymbols.insert_range(KV.second);

```
- **EN**: Implements logic around `JITDylib`, `JITLinkDylib`, `push_back`, `IL_removeTracker`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `JITDylib`, `JITLinkDylib`, `push_back`, `IL_removeTracker`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 1240-1259
```cpp
    for (auto &KV : Symbols) {
      auto &Sym = KV.first;
      if (!TrackedSymbols.count(Sym))
        SymbolsToRemove.push_back(Sym);
    }

    DefaultTracker.reset();
  } else {
    /// Check for a non-default tracker.
    auto I = TrackerSymbols.find(&RT);
    if (I != TrackerSymbols.end()) {
      SymbolsToRemove = std::move(I->second);
      TrackerSymbols.erase(I);
    }
    // ... if not found this tracker was already defunct. Nothing to do.
  }

  for (auto &Sym : SymbolsToRemove) {
    assert(Symbols.count(Sym) && "Symbol not in symbol table");

```
- **EN**: Implements logic around `push_back`, `reset`, `find`, `move`, and 2 more symbols.
- **CN**: 围绕 `push_back`, `reset`, `find`, `move`, and 2 more symbols 实现具体逻辑。

### Lines 1260-1289
```cpp
    // If there is a MaterializingInfo then collect any queries to fail.
    auto MII = MaterializingInfos.find(Sym);
    if (MII != MaterializingInfos.end())
      SymbolsToFail.push_back(Sym);
  }

  auto [QueriesToFail, FailedSymbols] =
      ES.IL_failSymbols(*this, std::move(SymbolsToFail));

  std::vector<std::unique_ptr<MaterializationUnit>> DefunctMUs;

  // Removed symbols should be taken out of the table altogether.
  for (auto &Sym : SymbolsToRemove) {
    auto I = Symbols.find(Sym);
    assert(I != Symbols.end() && "Symbol not present in table");

    // Remove Materializer if present.
    if (I->second.hasMaterializerAttached()) {
      // FIXME: Should this discard the symbols?
      auto J = UnmaterializedInfos.find(Sym);
      assert(J != UnmaterializedInfos.end() &&
             "Symbol table indicates MU present, but no UMI record");
      if (J->second->MU)
        DefunctMUs.push_back(std::move(J->second->MU));
      UnmaterializedInfos.erase(J);
    } else {
      assert(!UnmaterializedInfos.count(Sym) &&
             "Symbol has materializer attached");
    }

```
- **EN**: Implements logic around `find`, `push_back`, `IL_failSymbols`, `assert`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `find`, `push_back`, `IL_failSymbols`, `assert`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 1290-1310
```cpp
    Symbols.erase(I);
  }

  shrinkMaterializationInfoMemory();

  return {std::move(QueriesToFail), std::move(FailedSymbols),
          std::move(DefunctMUs)};
}

void JITDylib::transferTracker(ResourceTracker &DstRT, ResourceTracker &SrcRT) {
  assert(State != Closed && "JD is defunct");
  assert(&DstRT != &SrcRT && "No-op transfers shouldn't call transferTracker");
  assert(&DstRT.getJITDylib() == this && "DstRT is not for this JITDylib");
  assert(&SrcRT.getJITDylib() == this && "SrcRT is not for this JITDylib");

  // Update trackers for any not-yet materialized units.
  for (auto &KV : UnmaterializedInfos) {
    if (KV.second->RT == &SrcRT)
      KV.second->RT = &DstRT;
  }

```
- **EN**: Implements logic around `erase`, `shrinkMaterializationInfoMemory`, `move`, `transferTracker`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `erase`, `shrinkMaterializationInfoMemory`, `move`, `transferTracker`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 1311-1335
```cpp
  // Update trackers for any active materialization responsibilities.
  {
    auto I = TrackerMRs.find(&SrcRT);
    if (I != TrackerMRs.end()) {
      auto &SrcMRs = I->second;
      auto &DstMRs = TrackerMRs[&DstRT];
      for (auto *MR : SrcMRs)
        MR->RT = &DstRT;
      if (DstMRs.empty())
        DstMRs = std::move(SrcMRs);
      else
        DstMRs.insert_range(SrcMRs);
      // Erase SrcRT entry in TrackerMRs. Use &SrcRT key rather than iterator I
      // for this, since I may have been invalidated by 'TrackerMRs[&DstRT]'.
      TrackerMRs.erase(&SrcRT);
    }
  }

  // If we're transfering to the default tracker we just need to delete the
  // tracked symbols for the source tracker.
  if (&DstRT == DefaultTracker.get()) {
    TrackerSymbols.erase(&SrcRT);
    return;
  }

```
- **EN**: Implements logic around `find`, `move`, `insert_range`, `erase`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `find`, `move`, `insert_range`, `erase` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 1336-1357
```cpp
  // If we're transferring from the default tracker we need to find all
  // currently untracked symbols.
  if (&SrcRT == DefaultTracker.get()) {
    assert(!TrackerSymbols.count(&SrcRT) &&
           "Default tracker should not appear in TrackerSymbols");

    SymbolNameVector SymbolsToTrack;

    SymbolNameSet CurrentlyTrackedSymbols;
    for (auto &KV : TrackerSymbols)
      CurrentlyTrackedSymbols.insert_range(KV.second);

    for (auto &KV : Symbols) {
      auto &Sym = KV.first;
      if (!CurrentlyTrackedSymbols.count(Sym))
        SymbolsToTrack.push_back(Sym);
    }

    TrackerSymbols[&DstRT] = std::move(SymbolsToTrack);
    return;
  }

```
- **EN**: Implements logic around `assert`, `insert_range`, `push_back`, `move`.
- **CN**: 围绕 `assert`, `insert_range`, `push_back`, `move` 实现具体逻辑。

### Lines 1358-1378
```cpp
  auto &DstTrackedSymbols = TrackerSymbols[&DstRT];

  // Finally if neither SrtRT or DstRT are the default tracker then
  // just append DstRT's tracked symbols to SrtRT's.
  auto SI = TrackerSymbols.find(&SrcRT);
  if (SI == TrackerSymbols.end())
    return;

  DstTrackedSymbols.reserve(DstTrackedSymbols.size() + SI->second.size());
  for (auto &Sym : SI->second)
    DstTrackedSymbols.push_back(std::move(Sym));
  TrackerSymbols.erase(SI);
}

Error JITDylib::defineImpl(MaterializationUnit &MU) {
  LLVM_DEBUG({ dbgs() << "  " << MU.getSymbols() << "\n"; });

  SymbolNameSet Duplicates;
  std::vector<SymbolStringPtr> ExistingDefsOverridden;
  std::vector<SymbolStringPtr> MUDefsOverridden;

```
- **EN**: Implements logic around `find`, `reserve`, `push_back`, `erase`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `find`, `reserve`, `push_back`, `erase`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 1379-1405
```cpp
  for (const auto &KV : MU.getSymbols()) {
    auto I = Symbols.find(KV.first);

    if (I != Symbols.end()) {
      if (KV.second.isStrong()) {
        if (I->second.getFlags().isStrong() ||
            I->second.getState() > SymbolState::NeverSearched)
          Duplicates.insert(KV.first);
        else {
          assert(I->second.getState() == SymbolState::NeverSearched &&
                 "Overridden existing def should be in the never-searched "
                 "state");
          ExistingDefsOverridden.push_back(KV.first);
        }
      } else
        MUDefsOverridden.push_back(KV.first);
    }
  }

  // If there were any duplicate definitions then bail out.
  if (!Duplicates.empty()) {
    LLVM_DEBUG(
        { dbgs() << "  Error: Duplicate symbols " << Duplicates << "\n"; });
    return make_error<DuplicateDefinition>(std::string(**Duplicates.begin()),
                                           MU.getName().str());
  }

```
- **EN**: Implements logic around `find`, `getState`, `insert`, `assert`, and 4 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `getState`, `insert`, `assert`, and 4 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 1406-1427
```cpp
  // Discard any overridden defs in this MU.
  LLVM_DEBUG({
    if (!MUDefsOverridden.empty())
      dbgs() << "  Defs in this MU overridden: " << MUDefsOverridden << "\n";
  });
  for (auto &S : MUDefsOverridden)
    MU.doDiscard(*this, S);

  // Discard existing overridden defs.
  LLVM_DEBUG({
    if (!ExistingDefsOverridden.empty())
      dbgs() << "  Existing defs overridden by this MU: " << MUDefsOverridden
             << "\n";
  });
  for (auto &S : ExistingDefsOverridden) {

    auto UMII = UnmaterializedInfos.find(S);
    assert(UMII != UnmaterializedInfos.end() &&
           "Overridden existing def should have an UnmaterializedInfo");
    UMII->second->MU->doDiscard(*this, S);
  }

```
- **EN**: Implements logic around `dbgs`, `doDiscard`, `find`, `assert`.
- **CN**: 围绕 `dbgs`, `doDiscard`, `find`, `assert` 实现具体逻辑。

### Lines 1428-1449
```cpp
  // Finally, add the defs from this MU.
  for (auto &KV : MU.getSymbols()) {
    auto &SymEntry = Symbols[KV.first];
    SymEntry.setFlags(KV.second);
    SymEntry.setState(SymbolState::NeverSearched);
    SymEntry.setMaterializerAttached(true);
  }

  return Error::success();
}

void JITDylib::installMaterializationUnit(
    std::unique_ptr<MaterializationUnit> MU, ResourceTracker &RT) {

  /// defineImpl succeeded.
  if (&RT != DefaultTracker.get()) {
    auto &TS = TrackerSymbols[&RT];
    TS.reserve(TS.size() + MU->getSymbols().size());
    for (auto &KV : MU->getSymbols())
      TS.push_back(KV.first);
  }

```
- **EN**: Implements logic around `setFlags`, `setState`, `setMaterializerAttached`, `success`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `setFlags`, `setState`, `setMaterializerAttached`, `success`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 1450-1469
```cpp
  auto UMI = std::make_shared<UnmaterializedInfo>(std::move(MU), &RT);
  for (auto &KV : UMI->MU->getSymbols())
    UnmaterializedInfos[KV.first] = UMI;
}

void JITDylib::detachQueryHelper(AsynchronousSymbolQuery &Q,
                                 const SymbolNameSet &QuerySymbols) {
  for (auto &QuerySymbol : QuerySymbols) {
    auto MII = MaterializingInfos.find(QuerySymbol);
    if (MII != MaterializingInfos.end())
      MII->second.removeQuery(Q);
  }
}

Platform::~Platform() = default;

Expected<DenseMap<JITDylib *, SymbolMap>> Platform::lookupInitSymbols(
    ExecutionSession &ES,
    const DenseMap<JITDylib *, SymbolLookupSet> &InitSyms) {

```
- **EN**: Implements logic around `make_shared<UnmaterializedInfo>`, `detachQueryHelper`, `find`, `removeQuery`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `make_shared<UnmaterializedInfo>`, `detachQueryHelper`, `find`, `removeQuery`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 1470-1505
```cpp
  DenseMap<JITDylib *, SymbolMap> CompoundResult;
  Error CompoundErr = Error::success();
  std::mutex LookupMutex;
  std::condition_variable CV;
  uint64_t Count = InitSyms.size();

  LLVM_DEBUG({
    dbgs() << "Issuing init-symbol lookup:\n";
    for (auto &KV : InitSyms)
      dbgs() << "  " << KV.first->getName() << ": " << KV.second << "\n";
  });

  for (auto &KV : InitSyms) {
    auto *JD = KV.first;
    auto Names = std::move(KV.second);
    ES.lookup(
        LookupKind::Static,
        JITDylibSearchOrder({{JD, JITDylibLookupFlags::MatchAllSymbols}}),
        std::move(Names), SymbolState::Ready,
        [&, JD](Expected<SymbolMap> Result) {
          {
            std::lock_guard<std::mutex> Lock(LookupMutex);
            --Count;
            if (Result) {
              assert(!CompoundResult.count(JD) &&
                     "Duplicate JITDylib in lookup?");
              CompoundResult[JD] = std::move(*Result);
            } else
              CompoundErr =
                  joinErrors(std::move(CompoundErr), Result.takeError());
          }
          CV.notify_one();
        },
        NoDependenciesToRegister);
  }

```
- **EN**: Implements logic around `success`, `size`, `dbgs`, `move`, and 6 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `success`, `size`, `dbgs`, `move`, and 6 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 1506-1529
```cpp
  std::unique_lock<std::mutex> Lock(LookupMutex);
  CV.wait(Lock, [&] { return Count == 0; });

  if (CompoundErr)
    return std::move(CompoundErr);

  return std::move(CompoundResult);
}

void Platform::lookupInitSymbolsAsync(
    unique_function<void(Error)> OnComplete, ExecutionSession &ES,
    const DenseMap<JITDylib *, SymbolLookupSet> &InitSyms) {

  class TriggerOnComplete {
  public:
    using OnCompleteFn = unique_function<void(Error)>;
    TriggerOnComplete(OnCompleteFn OnComplete)
        : OnComplete(std::move(OnComplete)) {}
    ~TriggerOnComplete() { OnComplete(std::move(LookupResult)); }
    void reportResult(Error Err) {
      std::lock_guard<std::mutex> Lock(ResultMutex);
      LookupResult = joinErrors(std::move(LookupResult), std::move(Err));
    }

```
- **EN**: Introduces declarations for `TriggerOnComplete`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `TriggerOnComplete` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 1530-1557
```cpp
  private:
    std::mutex ResultMutex;
    Error LookupResult{Error::success()};
    OnCompleteFn OnComplete;
  };

  LLVM_DEBUG({
    dbgs() << "Issuing init-symbol lookup:\n";
    for (auto &KV : InitSyms)
      dbgs() << "  " << KV.first->getName() << ": " << KV.second << "\n";
  });

  auto TOC = std::make_shared<TriggerOnComplete>(std::move(OnComplete));

  for (auto &KV : InitSyms) {
    auto *JD = KV.first;
    auto Names = std::move(KV.second);
    ES.lookup(
        LookupKind::Static,
        JITDylibSearchOrder({{JD, JITDylibLookupFlags::MatchAllSymbols}}),
        std::move(Names), SymbolState::Ready,
        [TOC](Expected<SymbolMap> Result) {
          TOC->reportResult(Result.takeError());
        },
        NoDependenciesToRegister);
  }
}

```
- **EN**: Implements logic around `success`, `dbgs`, `make_shared<TriggerOnComplete>`, `move`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `success`, `dbgs`, `make_shared<TriggerOnComplete>`, `move`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 1558-1578
```cpp
MaterializationTask::~MaterializationTask() {
  // If this task wasn't run then fail materialization.
  if (MR)
    MR->failMaterialization();
}

void MaterializationTask::printDescription(raw_ostream &OS) {
  OS << "Materialization task: " << MU->getName() << " in "
     << MR->getTargetJITDylib().getName();
}

void MaterializationTask::run() {
  assert(MU && "MU should not be null");
  assert(MR && "MR should not be null");
  MU->materialize(std::move(MR));
}

void LookupTask::printDescription(raw_ostream &OS) { OS << "Lookup task"; }

void LookupTask::run() { LS.continueLookup(Error::success()); }

```
- **EN**: Implements logic around `~MaterializationTask`, `failMaterialization`, `printDescription`, `getName`, and 4 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `~MaterializationTask`, `failMaterialization`, `printDescription`, `getName`, and 4 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 1579-1600
```cpp
ExecutionSession::ExecutionSession(std::unique_ptr<ExecutorProcessControl> EPC)
    : EPC(std::move(EPC)), BootstrapJD(createBareJITDylib("<bootstrap>")) {
  // Associated EPC and this.
  this->EPC->ES = this;
  SymbolMap BootstrapSymbols;
  for (auto &[Name, Ptr] : this->EPC->getBootstrapSymbolsMap())
    BootstrapSymbols[intern(Name)] =
        ExecutorSymbolDef(Ptr, JITSymbolFlags::Exported);
  // Can't fail: BootstrapJD is a new, empty JD and the BootstrapSymbols
  // variable is a map, so can't contain duplicates.
  cantFail(BootstrapJD.define(absoluteSymbols(std::move(BootstrapSymbols))));
}

ExecutionSession::~ExecutionSession() {
  // You must call endSession prior to destroying the session.
  assert(!SessionOpen &&
         "Session still open. Did you forget to call endSession?");
}

Error ExecutionSession::endSession() {
  LLVM_DEBUG(dbgs() << "Ending ExecutionSession " << this << "\n");

```
- **EN**: Implements logic around `ExecutionSession`, `EPC`, `intern`, `ExecutorSymbolDef`, and 4 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `ExecutionSession`, `EPC`, `intern`, `ExecutorSymbolDef`, and 4 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 1601-1622
```cpp
  WaitingOnGraph::OpRecorder *GOpRecorderToEnd = nullptr;
  auto JDsToRemove = runSessionLocked([&] {

#ifdef EXPENSIVE_CHECKS
    verifySessionState("Entering ExecutionSession::endSession");
#endif

    if (SessionOpen)
      GOpRecorderToEnd = GOpRecorder;
    SessionOpen = false;
    return JDs;
  });

  std::reverse(JDsToRemove.begin(), JDsToRemove.end());

  auto Err = removeJITDylibs(std::move(JDsToRemove));

  Err = joinErrors(std::move(Err), EPC->disconnect());

  if (GOpRecorderToEnd)
    GOpRecorderToEnd->recordEnd();

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 1623-1642
```cpp
  return Err;
}

void ExecutionSession::registerResourceManager(ResourceManager &RM) {
  runSessionLocked([&] { ResourceManagers.push_back(&RM); });
}

void ExecutionSession::deregisterResourceManager(ResourceManager &RM) {
  runSessionLocked([&] {
    assert(!ResourceManagers.empty() && "No managers registered");
    if (ResourceManagers.back() == &RM)
      ResourceManagers.pop_back();
    else {
      auto I = llvm::find(ResourceManagers, &RM);
      assert(I != ResourceManagers.end() && "RM not registered");
      ResourceManagers.erase(I);
    }
  });
}

```
- **EN**: Implements logic around `registerResourceManager`, `runSessionLocked`, `deregisterResourceManager`, `assert`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `registerResourceManager`, `runSessionLocked`, `deregisterResourceManager`, `assert`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 1643-1668
```cpp
JITDylib *ExecutionSession::getJITDylibByName(StringRef Name) {
  return runSessionLocked([&, this]() -> JITDylib * {
    for (auto &JD : JDs)
      if (JD->getName() == Name)
        return JD.get();
    return nullptr;
  });
}

JITDylib &ExecutionSession::createBareJITDylib(std::string Name) {
  assert(!getJITDylibByName(Name) && "JITDylib with that name already exists");
  return runSessionLocked([&, this]() -> JITDylib & {
    assert(SessionOpen && "Cannot create JITDylib after session is closed");
    JDs.push_back(new JITDylib(*this, std::move(Name)));
    return *JDs.back();
  });
}

Expected<JITDylib &> ExecutionSession::createJITDylib(std::string Name) {
  auto &JD = createBareJITDylib(Name);
  if (P)
    if (auto Err = P->setupJITDylib(JD))
      return std::move(Err);
  return JD;
}

```
- **EN**: Implements logic around `getJITDylibByName`, `runSessionLocked`, `get`, `createBareJITDylib`, and 5 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `getJITDylibByName`, `runSessionLocked`, `get`, `createBareJITDylib`, and 5 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 1669-1688
```cpp
Error ExecutionSession::removeJITDylibs(std::vector<JITDylibSP> JDsToRemove) {
  // Set JD to 'Closing' state and remove JD from the ExecutionSession.
  runSessionLocked([&] {
    for (auto &JD : JDsToRemove) {
      assert(JD->State == JITDylib::Open && "JD already closed");
      JD->State = JITDylib::Closing;
      auto I = llvm::find(JDs, JD);
      assert(I != JDs.end() && "JD does not appear in session JDs");
      JDs.erase(I);
    }
  });

  // Clear JITDylibs and notify the platform.
  Error Err = Error::success();
  for (auto JD : JDsToRemove) {
    Err = joinErrors(std::move(Err), JD->clear());
    if (P)
      Err = joinErrors(std::move(Err), P->teardownJITDylib(*JD));
  }

```
- **EN**: Implements logic around `removeJITDylibs`, `runSessionLocked`, `assert`, `find`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `removeJITDylibs`, `runSessionLocked`, `assert`, `find`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 1689-1708
```cpp
  // Set JD to closed state. Clear remaining data structures.
  runSessionLocked([&] {
    for (auto &JD : JDsToRemove) {
      assert(JD->State == JITDylib::Closing && "JD should be closing");
      JD->State = JITDylib::Closed;
      assert(JD->Symbols.empty() && "JD.Symbols is not empty after clear");
      assert(JD->UnmaterializedInfos.empty() &&
             "JD.UnmaterializedInfos is not empty after clear");
      assert(JD->MaterializingInfos.empty() &&
             "JD.MaterializingInfos is not empty after clear");
      assert(JD->TrackerSymbols.empty() &&
             "TrackerSymbols is not empty after clear");
      JD->DefGenerators.clear();
      JD->LinkOrder.clear();
    }
  });

  return Err;
}

```
- **EN**: Implements logic around `runSessionLocked`, `assert`, `clear`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `runSessionLocked`, `assert`, `clear` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 1709-1731
```cpp
Expected<std::vector<JITDylibSP>>
JITDylib::getDFSLinkOrder(ArrayRef<JITDylibSP> JDs) {
  if (JDs.empty())
    return std::vector<JITDylibSP>();

  auto &ES = JDs.front()->getExecutionSession();
  return ES.runSessionLocked([&]() -> Expected<std::vector<JITDylibSP>> {
    DenseSet<JITDylib *> Visited;
    std::vector<JITDylibSP> Result;

    for (auto &JD : JDs) {

      if (JD->State != Open)
        return make_error<StringError>(
            "Error building link order: " + JD->getName() + " is defunct",
            inconvertibleErrorCode());
      if (Visited.count(JD.get()))
        continue;

      SmallVector<JITDylibSP, 64> WorkStack;
      WorkStack.push_back(JD);
      Visited.insert(JD.get());

```
- **EN**: Implements logic around `getDFSLinkOrder`, `vector<JITDylibSP>`, `front`, `runSessionLocked`, and 5 more symbols; this block coordinates ORC symbol lookup or materialization state; executes or prepares runtime behavior for LLVM IR/JIT code; returns subsystem-specific computed results.
- **CN**: 围绕 `getDFSLinkOrder`, `vector<JITDylibSP>`, `front`, `runSessionLocked`, and 5 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，执行或准备 LLVM IR/JIT 代码的运行时行为，返回子系统相关的计算结果。

### Lines 1732-1755
```cpp
      while (!WorkStack.empty()) {
        Result.push_back(std::move(WorkStack.back()));
        WorkStack.pop_back();

        for (auto &KV : llvm::reverse(Result.back()->LinkOrder)) {
          auto &JD = *KV.first;
          if (!Visited.insert(&JD).second)
            continue;
          WorkStack.push_back(&JD);
        }
      }
    }
    return Result;
  });
}

Expected<std::vector<JITDylibSP>>
JITDylib::getReverseDFSLinkOrder(ArrayRef<JITDylibSP> JDs) {
  auto Result = getDFSLinkOrder(JDs);
  if (Result)
    std::reverse(Result->begin(), Result->end());
  return Result;
}

```
- **EN**: Implements logic around `push_back`, `pop_back`, `getReverseDFSLinkOrder`, `getDFSLinkOrder`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state; executes or prepares runtime behavior for LLVM IR/JIT code; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `pop_back`, `getReverseDFSLinkOrder`, `getDFSLinkOrder`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，执行或准备 LLVM IR/JIT 代码的运行时行为，返回子系统相关的计算结果。

### Lines 1756-1777
```cpp
Expected<std::vector<JITDylibSP>> JITDylib::getDFSLinkOrder() {
  return getDFSLinkOrder({this});
}

Expected<std::vector<JITDylibSP>> JITDylib::getReverseDFSLinkOrder() {
  return getReverseDFSLinkOrder({this});
}

void ExecutionSession::lookupFlags(
    LookupKind K, JITDylibSearchOrder SearchOrder, SymbolLookupSet LookupSet,
    unique_function<void(Expected<SymbolFlagsMap>)> OnComplete) {

  OL_applyQueryPhase1(std::make_unique<InProgressLookupFlagsState>(
                          K, std::move(SearchOrder), std::move(LookupSet),
                          std::move(OnComplete)),
                      Error::success());
}

Expected<SymbolFlagsMap>
ExecutionSession::lookupFlags(LookupKind K, JITDylibSearchOrder SearchOrder,
                              SymbolLookupSet LookupSet) {

```
- **EN**: Implements logic around `getDFSLinkOrder`, `getReverseDFSLinkOrder`, `lookupFlags`, `unique_function<void`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `getDFSLinkOrder`, `getReverseDFSLinkOrder`, `lookupFlags`, `unique_function<void`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 1778-1802
```cpp
  std::promise<MSVCPExpected<SymbolFlagsMap>> ResultP;
  OL_applyQueryPhase1(std::make_unique<InProgressLookupFlagsState>(
                          K, std::move(SearchOrder), std::move(LookupSet),
                          [&ResultP](Expected<SymbolFlagsMap> Result) {
                            ResultP.set_value(std::move(Result));
                          }),
                      Error::success());

  auto ResultF = ResultP.get_future();
  return ResultF.get();
}

void ExecutionSession::lookup(
    LookupKind K, const JITDylibSearchOrder &SearchOrder,
    SymbolLookupSet Symbols, SymbolState RequiredState,
    SymbolsResolvedCallback NotifyComplete,
    RegisterDependenciesFunction RegisterDependencies) {

  LLVM_DEBUG({
    runSessionLocked([&]() {
      dbgs() << "Looking up " << Symbols << " in " << SearchOrder
             << " (required state: " << RequiredState << ")\n";
    });
  });

```
- **EN**: Implements logic around `OL_applyQueryPhase1`, `move`, `set_value`, `success`, and 5 more symbols; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `OL_applyQueryPhase1`, `move`, `set_value`, `success`, and 5 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 1803-1827
```cpp
  // lookup can be re-entered recursively if running on a single thread. Run any
  // outstanding MUs in case this query depends on them, otherwise this lookup
  // will starve waiting for a result from an MU that is stuck in the queue.
  dispatchOutstandingMUs();

  auto Unresolved = std::move(Symbols);
  auto Q = std::make_shared<AsynchronousSymbolQuery>(Unresolved, RequiredState,
                                                     std::move(NotifyComplete));

  auto IPLS = std::make_unique<InProgressFullLookupState>(
      K, SearchOrder, std::move(Unresolved), RequiredState, std::move(Q),
      std::move(RegisterDependencies));

  OL_applyQueryPhase1(std::move(IPLS), Error::success());
}

Expected<SymbolMap>
ExecutionSession::lookup(const JITDylibSearchOrder &SearchOrder,
                         SymbolLookupSet Symbols, LookupKind K,
                         SymbolState RequiredState,
                         RegisterDependenciesFunction RegisterDependencies) {
#if LLVM_ENABLE_THREADS
  // In the threaded case we use promises to return the results.
  std::promise<MSVCPExpected<SymbolMap>> PromisedResult;

```
- **EN**: Implements logic around `dispatchOutstandingMUs`, `move`, `make_shared<AsynchronousSymbolQuery>`, `make_unique<InProgressFullLookupState>`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `dispatchOutstandingMUs`, `move`, `make_shared<AsynchronousSymbolQuery>`, `make_unique<InProgressFullLookupState>`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 1828-1848
```cpp
  auto NotifyComplete = [&](Expected<SymbolMap> R) {
    PromisedResult.set_value(std::move(R));
  };

#else
  SymbolMap Result;
  Error ResolutionError = Error::success();

  auto NotifyComplete = [&](Expected<SymbolMap> R) {
    ErrorAsOutParameter _(ResolutionError);
    if (R)
      Result = std::move(*R);
    else
      ResolutionError = R.takeError();
  };
#endif

  // Perform the asynchronous lookup.
  lookup(K, SearchOrder, std::move(Symbols), RequiredState,
         std::move(NotifyComplete), RegisterDependencies);

```
- **EN**: Implements logic around `set_value`, `success`, `_`, `move`, and 2 more symbols.
- **CN**: 围绕 `set_value`, `success`, `_`, `move`, and 2 more symbols 实现具体逻辑。

### Lines 1849-1872
```cpp
#if LLVM_ENABLE_THREADS
  return PromisedResult.get_future().get();
#else
  if (ResolutionError)
    return std::move(ResolutionError);

  return Result;
#endif
}

Expected<ExecutorSymbolDef>
ExecutionSession::lookup(const JITDylibSearchOrder &SearchOrder,
                         SymbolStringPtr Name, SymbolState RequiredState) {
  SymbolLookupSet Names({Name});

  if (auto ResultMap = lookup(SearchOrder, std::move(Names), LookupKind::Static,
                              RequiredState, NoDependenciesToRegister)) {
    assert(ResultMap->size() == 1 && "Unexpected number of results");
    assert(ResultMap->count(Name) && "Missing result for symbol");
    return std::move(ResultMap->begin()->second);
  } else
    return ResultMap.takeError();
}

```
- **EN**: Implements logic around `get_future`, `move`, `lookup`, `Names`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `get_future`, `move`, `lookup`, `Names`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 1873-1893
```cpp
Expected<ExecutorSymbolDef>
ExecutionSession::lookup(ArrayRef<JITDylib *> SearchOrder, SymbolStringPtr Name,
                         SymbolState RequiredState) {
  return lookup(makeJITDylibSearchOrder(SearchOrder), Name, RequiredState);
}

Expected<ExecutorSymbolDef>
ExecutionSession::lookup(ArrayRef<JITDylib *> SearchOrder, StringRef Name,
                         SymbolState RequiredState) {
  return lookup(SearchOrder, intern(Name), RequiredState);
}

Error ExecutionSession::registerJITDispatchHandlers(
    JITDylib &JD, JITDispatchHandlerAssociationMap WFs) {

  auto TagSyms = lookup({{&JD, JITDylibLookupFlags::MatchAllSymbols}},
                        SymbolLookupSet::fromMapKeys(
                            WFs, SymbolLookupFlags::WeaklyReferencedSymbol));
  if (!TagSyms)
    return TagSyms.takeError();

```
- **EN**: Implements logic around `lookup`, `registerJITDispatchHandlers`, `fromMapKeys`, `takeError`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `lookup`, `registerJITDispatchHandlers`, `fromMapKeys`, `takeError` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 1894-1920
```cpp
  // Associate tag addresses with implementations.
  std::lock_guard<std::mutex> Lock(JITDispatchHandlersMutex);

  // Check that no tags are being overwritten.
  for (auto &[TagName, TagSym] : *TagSyms) {
    auto TagAddr = TagSym.getAddress();
    if (JITDispatchHandlers.count(TagAddr))
      return make_error<StringError>("Tag " + formatv("{0:x}", TagAddr) +
                                         " (for " + *TagName +
                                         ") already registered",
                                     inconvertibleErrorCode());
  }

  // At this point we're guaranteed to succeed. Install the handlers.
  for (auto &[TagName, TagSym] : *TagSyms) {
    auto TagAddr = TagSym.getAddress();
    auto I = WFs.find(TagName);
    assert(I != WFs.end() && I->second &&
           "JITDispatchHandler implementation missing");
    JITDispatchHandlers[TagAddr] =
        std::make_shared<JITDispatchHandlerFunction>(std::move(I->second));
    LLVM_DEBUG({
      dbgs() << "Associated function tag \"" << *TagName << "\" ("
             << formatv("{0:x}", TagAddr) << ") with handler\n";
    });
  }

```
- **EN**: Implements logic around `Lock`, `getAddress`, `make_error<StringError>`, `inconvertibleErrorCode`, and 5 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `Lock`, `getAddress`, `make_error<StringError>`, `inconvertibleErrorCode`, and 5 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 1921-1944
```cpp
  return Error::success();
}

void ExecutionSession::runJITDispatchHandler(
    SendResultFunction SendResult, ExecutorAddr HandlerFnTagAddr,
    shared::WrapperFunctionBuffer ArgBytes) {

  std::shared_ptr<JITDispatchHandlerFunction> F;
  {
    std::lock_guard<std::mutex> Lock(JITDispatchHandlersMutex);
    auto I = JITDispatchHandlers.find(HandlerFnTagAddr);
    if (I != JITDispatchHandlers.end())
      F = I->second;
  }

  if (F)
    (*F)(std::move(SendResult), ArgBytes.data(), ArgBytes.size());
  else
    SendResult(shared::WrapperFunctionBuffer::createOutOfBandError(
        ("No function registered for tag " +
         formatv("{0:x16}", HandlerFnTagAddr))
            .str()));
}

```
- **EN**: Implements logic around `success`, `runJITDispatchHandler`, `Lock`, `find`, and 4 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `runJITDispatchHandler`, `Lock`, `find`, and 4 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 1945-1968
```cpp
void ExecutionSession::dump(raw_ostream &OS) {
  runSessionLocked([this, &OS]() {
    for (auto &JD : JDs)
      JD->dump(OS);
  });
}

#ifdef EXPENSIVE_CHECKS
bool ExecutionSession::verifySessionState(Twine Phase) {
  return runSessionLocked([&]() {
    bool AllOk = true;

    for (auto &JD : JDs) {

      auto LogFailure = [&]() -> raw_fd_ostream & {
        auto &Stream = errs();
        if (AllOk)
          Stream << "ERROR: Bad ExecutionSession state detected " << Phase
                 << "\n";
        Stream << "  In JITDylib " << JD->getName() << ", ";
        AllOk = false;
        return Stream;
      };

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 1969-1989
```cpp
      if (JD->State != JITDylib::Open) {
        LogFailure()
            << "state is not Open, but JD is in ExecutionSession list.";
      }

      // Check symbol table.
      // 1. If the entry state isn't resolved then check that no address has
      //    been set.
      // 2. Check that if the hasMaterializerAttached flag is set then there is
      //    an UnmaterializedInfo entry, and vice-versa.
      for (auto &[Sym, Entry] : JD->Symbols) {
        // Check that unresolved symbols have null addresses.
        if (Entry.getState() < SymbolState::Resolved) {
          if (Entry.getAddress()) {
            LogFailure() << "symbol " << Sym << " has state "
                         << Entry.getState()
                         << " (not-yet-resolved) but non-null address "
                         << Entry.getAddress() << ".\n";
          }
        }

```
- **EN**: Implements logic around `LogFailure`, `getState`, `getAddress`; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `LogFailure`, `getState`, `getAddress` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作。

### Lines 1990-2016
```cpp
        // Check that the hasMaterializerAttached flag is correct.
        auto UMIItr = JD->UnmaterializedInfos.find(Sym);
        if (Entry.hasMaterializerAttached()) {
          if (UMIItr == JD->UnmaterializedInfos.end()) {
            LogFailure() << "symbol " << Sym
                         << " entry claims materializer attached, but "
                            "UnmaterializedInfos has no corresponding entry.\n";
          }
        } else if (UMIItr != JD->UnmaterializedInfos.end()) {
          LogFailure()
              << "symbol " << Sym
              << " entry claims no materializer attached, but "
                 "UnmaterializedInfos has an unexpected entry for it.\n";
        }
      }

      // Check that every UnmaterializedInfo entry has a corresponding entry
      // in the Symbols table.
      for (auto &[Sym, UMI] : JD->UnmaterializedInfos) {
        auto SymItr = JD->Symbols.find(Sym);
        if (SymItr == JD->Symbols.end()) {
          LogFailure()
              << "symbol " << Sym
              << " has UnmaterializedInfos entry, but no Symbols entry.\n";
        }
      }

```
- **EN**: Implements logic around `find`, `LogFailure`.
- **CN**: 围绕 `find`, `LogFailure` 实现具体逻辑。

### Lines 2017-2036
```cpp
      // Check consistency of the MaterializingInfos table.
      for (auto &[Sym, MII] : JD->MaterializingInfos) {

        auto SymItr = JD->Symbols.find(Sym);
        if (SymItr == JD->Symbols.end()) {
          // If there's no Symbols entry for this MaterializingInfos entry then
          // report that.
          LogFailure()
              << "symbol " << Sym
              << " has MaterializingInfos entry, but no Symbols entry.\n";
        } else {
          // Otherwise check consistency between Symbols and MaterializingInfos.

          // Ready symbols should not have MaterializingInfos.
          if (SymItr->second.getState() == SymbolState::Ready) {
            LogFailure()
                << "symbol " << Sym
                << " is in Ready state, should not have MaterializingInfo.\n";
          }

```
- **EN**: Implements logic around `find`, `LogFailure`.
- **CN**: 围绕 `find`, `LogFailure` 实现具体逻辑。

### Lines 2037-2058
```cpp
          // Pending queries should be for subsequent states.
          auto CurState = static_cast<SymbolState>(
              static_cast<std::underlying_type_t<SymbolState>>(
                  SymItr->second.getState()) + 1);
          for (auto &Q : MII.PendingQueries) {
            if (Q->getRequiredState() != CurState) {
              if (Q->getRequiredState() > CurState)
                CurState = Q->getRequiredState();
              else
                LogFailure() << "symbol " << Sym
                             << " has stale or misordered queries.\n";
            }
          }
        }
      }
    }

    return AllOk;
  });
}
#endif // EXPENSIVE_CHECKS

```
- **EN**: Implements logic around `static_cast<SymbolState>`, `underlying_type_t<SymbolState>>`, `getState`, `getRequiredState`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `static_cast<SymbolState>`, `underlying_type_t<SymbolState>>`, `getState`, `getRequiredState`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 2059-2084
```cpp
void ExecutionSession::dispatchOutstandingMUs() {
  LLVM_DEBUG(dbgs() << "Dispatching MaterializationUnits...\n");
  while (true) {
    std::optional<std::pair<std::unique_ptr<MaterializationUnit>,
                            std::unique_ptr<MaterializationResponsibility>>>
        JMU;

    {
      std::lock_guard<std::recursive_mutex> Lock(OutstandingMUsMutex);
      if (!OutstandingMUs.empty()) {
        JMU.emplace(std::move(OutstandingMUs.back()));
        OutstandingMUs.pop_back();
      }
    }

    if (!JMU)
      break;

    assert(JMU->first && "No MU?");
    LLVM_DEBUG(dbgs() << "  Dispatching \"" << JMU->first->getName() << "\"\n");
    dispatchTask(std::make_unique<MaterializationTask>(std::move(JMU->first),
                                                       std::move(JMU->second)));
  }
  LLVM_DEBUG(dbgs() << "Done dispatching MaterializationUnits.\n");
}

```
- **EN**: Implements logic around `dispatchOutstandingMUs`, `Lock`, `emplace`, `pop_back`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `dispatchOutstandingMUs`, `Lock`, `emplace`, `pop_back`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 2085-2104
```cpp
Error ExecutionSession::removeResourceTracker(ResourceTracker &RT) {
  LLVM_DEBUG({
    dbgs() << "In " << RT.getJITDylib().getName() << " removing tracker "
           << formatv("{0:x}", RT.getKeyUnsafe()) << "\n";
  });
  std::vector<ResourceManager *> CurrentResourceManagers;

  JITDylib::RemoveTrackerResult R;

  runSessionLocked([&] {
    CurrentResourceManagers = ResourceManagers;
    RT.makeDefunct();
    R = RT.getJITDylib().IL_removeTracker(RT);
  });

  // Release any defunct MaterializationUnits.
  R.DefunctMUs.clear();

  Error Err = Error::success();

```
- **EN**: Implements logic around `removeResourceTracker`, `dbgs`, `formatv`, `runSessionLocked`, and 4 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `removeResourceTracker`, `dbgs`, `formatv`, `runSessionLocked`, and 4 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 2105-2125
```cpp
  auto &JD = RT.getJITDylib();
  for (auto *L : reverse(CurrentResourceManagers))
    Err = joinErrors(std::move(Err),
                     L->handleRemoveResources(JD, RT.getKeyUnsafe()));

  for (auto &Q : R.QueriesToFail)
    Q->handleFailed(make_error<FailedToMaterialize>(getSymbolStringPool(),
                                                    R.FailedSymbols));

  return Err;
}

void ExecutionSession::transferResourceTracker(ResourceTracker &DstRT,
                                               ResourceTracker &SrcRT) {
  LLVM_DEBUG({
    dbgs() << "In " << SrcRT.getJITDylib().getName()
           << " transfering resources from tracker "
           << formatv("{0:x}", SrcRT.getKeyUnsafe()) << " to tracker "
           << formatv("{0:x}", DstRT.getKeyUnsafe()) << "\n";
  });

```
- **EN**: Implements logic around `getJITDylib`, `joinErrors`, `handleRemoveResources`, `handleFailed`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `getJITDylib`, `joinErrors`, `handleRemoveResources`, `handleFailed`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 2126-2153
```cpp
  // No-op transfers are allowed and do not invalidate the source.
  if (&DstRT == &SrcRT)
    return;

  assert(&DstRT.getJITDylib() == &SrcRT.getJITDylib() &&
         "Can't transfer resources between JITDylibs");
  runSessionLocked([&]() {
    SrcRT.makeDefunct();
    auto &JD = DstRT.getJITDylib();
    JD.transferTracker(DstRT, SrcRT);
    for (auto *L : reverse(ResourceManagers))
      L->handleTransferResources(JD, DstRT.getKeyUnsafe(),
                                 SrcRT.getKeyUnsafe());
  });
}

void ExecutionSession::destroyResourceTracker(ResourceTracker &RT) {
  runSessionLocked([&]() {
    LLVM_DEBUG({
      dbgs() << "In " << RT.getJITDylib().getName() << " destroying tracker "
             << formatv("{0:x}", RT.getKeyUnsafe()) << "\n";
    });
    if (!RT.isDefunct())
      transferResourceTracker(*RT.getJITDylib().getDefaultResourceTracker(),
                              RT);
  });
}

```
- **EN**: Implements logic around `assert`, `runSessionLocked`, `makeDefunct`, `getJITDylib`, and 7 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `assert`, `runSessionLocked`, `makeDefunct`, `getJITDylib`, and 7 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 2154-2177
```cpp
Error ExecutionSession::IL_updateCandidatesFor(
    JITDylib &JD, JITDylibLookupFlags JDLookupFlags,
    SymbolLookupSet &Candidates, SymbolLookupSet *NonCandidates) {
  return Candidates.forEachWithRemoval(
      [&](const SymbolStringPtr &Name,
          SymbolLookupFlags SymLookupFlags) -> Expected<bool> {
        /// Search for the symbol. If not found then continue without
        /// removal.
        auto SymI = JD.Symbols.find(Name);
        if (SymI == JD.Symbols.end())
          return false;

        // If this is a non-exported symbol and we're matching exported
        // symbols only then remove this symbol from the candidates list.
        //
        // If we're tracking non-candidates then add this to the non-candidate
        // list.
        if (!SymI->second.getFlags().isExported() &&
            JDLookupFlags == JITDylibLookupFlags::MatchExportedSymbolsOnly) {
          if (NonCandidates)
            NonCandidates->add(Name, SymLookupFlags);
          return true;
        }

```
- **EN**: Implements logic around `IL_updateCandidatesFor`, `forEachWithRemoval`, `find`, `add`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `IL_updateCandidatesFor`, `forEachWithRemoval`, `find`, `add` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 2178-2201
```cpp
        // If we match against a materialization-side-effects only symbol
        // then make sure it is weakly-referenced. Otherwise bail out with
        // an error.
        // FIXME: Use a "materialization-side-effects-only symbols must be
        // weakly referenced" specific error here to reduce confusion.
        if (SymI->second.getFlags().hasMaterializationSideEffectsOnly() &&
            SymLookupFlags != SymbolLookupFlags::WeaklyReferencedSymbol)
          return make_error<SymbolsNotFound>(getSymbolStringPool(),
                                             SymbolNameVector({Name}));

        // If we matched against this symbol but it is in the error state
        // then bail out and treat it as a failure to materialize.
        if (SymI->second.getFlags().hasError()) {
          auto FailedSymbolsMap = std::make_shared<SymbolDependenceMap>();
          (*FailedSymbolsMap)[&JD] = {Name};
          return make_error<FailedToMaterialize>(getSymbolStringPool(),
                                                 std::move(FailedSymbolsMap));
        }

        // Otherwise this is a match. Remove it from the candidate set.
        return true;
      });
}

```
- **EN**: Implements logic around `make_error<SymbolsNotFound>`, `SymbolNameVector`, `make_shared<SymbolDependenceMap>`, `make_error<FailedToMaterialize>`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<SymbolsNotFound>`, `SymbolNameVector`, `make_shared<SymbolDependenceMap>`, `make_error<FailedToMaterialize>`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 2202-2221
```cpp
void ExecutionSession::OL_resumeLookupAfterGeneration(
    InProgressLookupState &IPLS) {

  assert(IPLS.GenState != InProgressLookupState::NotInGenerator &&
         "Should not be called for not-in-generator lookups");
  IPLS.GenState = InProgressLookupState::NotInGenerator;

  LookupState LS;

  if (auto DG = IPLS.CurDefGeneratorStack.back().lock()) {
    IPLS.CurDefGeneratorStack.pop_back();
    std::lock_guard<std::mutex> Lock(DG->M);

    // If there are no pending lookups then mark the generator as free and
    // return.
    if (DG->PendingLookups.empty()) {
      DG->InUse = false;
      return;
    }

```
- **EN**: Implements logic around `OL_resumeLookupAfterGeneration`, `assert`, `pop_back`, `Lock`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `OL_resumeLookupAfterGeneration`, `assert`, `pop_back`, `Lock` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 2222-2248
```cpp
    // Otherwise resume the next lookup.
    LS = std::move(DG->PendingLookups.front());
    DG->PendingLookups.pop_front();
  }

  if (LS.IPLS) {
    LS.IPLS->GenState = InProgressLookupState::ResumedForGenerator;
    dispatchTask(std::make_unique<LookupTask>(std::move(LS)));
  }
}

void ExecutionSession::OL_applyQueryPhase1(
    std::unique_ptr<InProgressLookupState> IPLS, Error Err) {

  LLVM_DEBUG({
    dbgs() << "Entering OL_applyQueryPhase1:\n"
           << "  Lookup kind: " << IPLS->K << "\n"
           << "  Search order: " << IPLS->SearchOrder
           << ", Current index = " << IPLS->CurSearchOrderIndex
           << (IPLS->NewJITDylib ? " (entering new JITDylib)" : "") << "\n"
           << "  Lookup set: " << IPLS->LookupSet << "\n"
           << "  Definition generator candidates: "
           << IPLS->DefGeneratorCandidates << "\n"
           << "  Definition generator non-candidates: "
           << IPLS->DefGeneratorNonCandidates << "\n";
  });

```
- **EN**: Implements logic around `move`, `pop_front`, `dispatchTask`, `OL_applyQueryPhase1`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `move`, `pop_front`, `dispatchTask`, `OL_applyQueryPhase1`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 2249-2272
```cpp
  if (IPLS->GenState == InProgressLookupState::InGenerator)
    OL_resumeLookupAfterGeneration(*IPLS);

  assert(IPLS->GenState != InProgressLookupState::InGenerator &&
         "Lookup should not be in InGenerator state here");

  // FIXME: We should attach the query as we go: This provides a result in a
  // single pass in the common case where all symbols have already reached the
  // required state. The query could be detached again in the 'fail' method on
  // IPLS. Phase 2 would be reduced to collecting and dispatching the MUs.

  while (IPLS->CurSearchOrderIndex != IPLS->SearchOrder.size()) {

    // If we've been handed an error or received one back from a generator then
    // fail the query. We don't need to unlink: At this stage the query hasn't
    // actually been lodged.
    if (Err)
      return IPLS->fail(std::move(Err));

    // Get the next JITDylib and lookup flags.
    auto &KV = IPLS->SearchOrder[IPLS->CurSearchOrderIndex];
    auto &JD = *KV.first;
    auto JDLookupFlags = KV.second;

```
- **EN**: Implements logic around `OL_resumeLookupAfterGeneration`, `assert`, `fail`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `OL_resumeLookupAfterGeneration`, `assert`, `fail` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 2273-2298
```cpp
    LLVM_DEBUG({
      dbgs() << "Visiting \"" << JD.getName() << "\" (" << JDLookupFlags
             << ") with lookup set " << IPLS->LookupSet << ":\n";
    });

    // If we've just reached a new JITDylib then perform some setup.
    if (IPLS->NewJITDylib) {
      // Add any non-candidates from the last JITDylib (if any) back on to the
      // list of definition candidates for this JITDylib, reset definition
      // non-candidates to the empty set.
      SymbolLookupSet Tmp;
      std::swap(IPLS->DefGeneratorNonCandidates, Tmp);
      IPLS->DefGeneratorCandidates.append(std::move(Tmp));

      LLVM_DEBUG({
        dbgs() << "  First time visiting " << JD.getName()
               << ", resetting candidate sets and building generator stack\n";
      });

      // Build the definition generator stack for this JITDylib.
      runSessionLocked([&] {
        IPLS->CurDefGeneratorStack.reserve(JD.DefGenerators.size());
        llvm::append_range(IPLS->CurDefGeneratorStack,
                           reverse(JD.DefGenerators));
      });

```
- **EN**: Implements logic around `dbgs`, `swap`, `append`, `runSessionLocked`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `dbgs`, `swap`, `append`, `runSessionLocked`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 2299-2325
```cpp
      // Flag that we've done our initialization.
      IPLS->NewJITDylib = false;
    }

    // Remove any generation candidates that are already defined (and match) in
    // this JITDylib.
    runSessionLocked([&] {
      // Update the list of candidates (and non-candidates) for definition
      // generation.
      LLVM_DEBUG(dbgs() << "  Updating candidate set...\n");
      Err = IL_updateCandidatesFor(
          JD, JDLookupFlags, IPLS->DefGeneratorCandidates,
          JD.DefGenerators.empty() ? nullptr
                                   : &IPLS->DefGeneratorNonCandidates);
      LLVM_DEBUG({
        dbgs() << "    Remaining candidates = " << IPLS->DefGeneratorCandidates
               << "\n";
      });

      // If this lookup was resumed after auto-suspension but all candidates
      // have already been generated (by some previous call to the generator)
      // treat the lookup as if it had completed generation.
      if (IPLS->GenState == InProgressLookupState::ResumedForGenerator &&
          IPLS->DefGeneratorCandidates.empty())
        OL_resumeLookupAfterGeneration(*IPLS);
    });

```
- **EN**: Implements logic around `runSessionLocked`, `IL_updateCandidatesFor`, `empty`, `dbgs`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `runSessionLocked`, `IL_updateCandidatesFor`, `empty`, `dbgs`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 2326-2345
```cpp
    // If we encountered an error while filtering generation candidates then
    // bail out.
    if (Err)
      return IPLS->fail(std::move(Err));

    /// Apply any definition generators on the stack.
    LLVM_DEBUG({
      if (IPLS->CurDefGeneratorStack.empty())
        LLVM_DEBUG(dbgs() << "  No generators to run for this JITDylib.\n");
      else if (IPLS->DefGeneratorCandidates.empty())
        LLVM_DEBUG(dbgs() << "  No candidates to generate.\n");
      else
        dbgs() << "  Running " << IPLS->CurDefGeneratorStack.size()
               << " remaining generators for "
               << IPLS->DefGeneratorCandidates.size() << " candidates\n";
    });
    while (!IPLS->CurDefGeneratorStack.empty() &&
           !IPLS->DefGeneratorCandidates.empty()) {
      auto DG = IPLS->CurDefGeneratorStack.back().lock();

```
- **EN**: Implements logic around `fail`, `dbgs`, `size`, `empty`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `fail`, `dbgs`, `size`, `empty`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 2346-2368
```cpp
      if (!DG)
        return IPLS->fail(make_error<StringError>(
            "DefinitionGenerator removed while lookup in progress",
            inconvertibleErrorCode()));

      // At this point the lookup is in either the NotInGenerator state, or in
      // the ResumedForGenerator state.
      // If this lookup is in the NotInGenerator state then check whether the
      // generator is in use. If the generator is not in use then move the
      // lookup to the InGenerator state and continue. If the generator is
      // already in use then just add this lookup to the pending lookups list
      // and bail out.
      // If this lookup is in the ResumedForGenerator state then just move it
      // to InGenerator and continue.
      if (IPLS->GenState == InProgressLookupState::NotInGenerator) {
        std::lock_guard<std::mutex> Lock(DG->M);
        if (DG->InUse) {
          DG->PendingLookups.push_back(std::move(IPLS));
          return;
        }
        DG->InUse = true;
      }

```
- **EN**: Implements logic around `fail`, `inconvertibleErrorCode`, `Lock`, `push_back`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `fail`, `inconvertibleErrorCode`, `Lock`, `push_back` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 2369-2396
```cpp
      IPLS->GenState = InProgressLookupState::InGenerator;

      auto K = IPLS->K;
      auto &LookupSet = IPLS->DefGeneratorCandidates;

      // Run the generator. If the generator takes ownership of QA then this
      // will break the loop.
      {
        LLVM_DEBUG(dbgs() << "  Attempting to generate " << LookupSet << "\n");
        LookupState LS(std::move(IPLS));
        Err = DG->tryToGenerate(LS, K, JD, JDLookupFlags, LookupSet);
        IPLS = std::move(LS.IPLS);
      }

      // If the lookup returned then pop the generator stack and unblock the
      // next lookup on this generator (if any).
      if (IPLS)
        OL_resumeLookupAfterGeneration(*IPLS);

      // If there was an error then fail the query.
      if (Err) {
        LLVM_DEBUG({
          dbgs() << "  Error attempting to generate " << LookupSet << "\n";
        });
        assert(IPLS && "LS cannot be retained if error is returned");
        return IPLS->fail(std::move(Err));
      }

```
- **EN**: Implements logic around `LS`, `tryToGenerate`, `move`, `OL_resumeLookupAfterGeneration`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `LS`, `tryToGenerate`, `move`, `OL_resumeLookupAfterGeneration`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 2397-2420
```cpp
      // Otherwise if QA was captured then break the loop.
      if (!IPLS) {
        LLVM_DEBUG(
            { dbgs() << "  LookupState captured. Exiting phase1 for now.\n"; });
        return;
      }

      // Otherwise if we're continuing around the loop then update candidates
      // for the next round.
      runSessionLocked([&] {
        LLVM_DEBUG(dbgs() << "  Updating candidate set post-generation\n");
        Err = IL_updateCandidatesFor(
            JD, JDLookupFlags, IPLS->DefGeneratorCandidates,
            JD.DefGenerators.empty() ? nullptr
                                     : &IPLS->DefGeneratorNonCandidates);
      });

      // If updating candidates failed then fail the query.
      if (Err) {
        LLVM_DEBUG(dbgs() << "  Error encountered while updating candidates\n");
        return IPLS->fail(std::move(Err));
      }
    }

```
- **EN**: Implements logic around `dbgs`, `runSessionLocked`, `IL_updateCandidatesFor`, `empty`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `runSessionLocked`, `IL_updateCandidatesFor`, `empty`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 2421-2441
```cpp
    if (IPLS->DefGeneratorCandidates.empty() &&
        IPLS->DefGeneratorNonCandidates.empty()) {
      // Early out if there are no remaining symbols.
      LLVM_DEBUG(dbgs() << "All symbols matched.\n");
      IPLS->CurSearchOrderIndex = IPLS->SearchOrder.size();
      break;
    } else {
      // If we get here then we've moved on to the next JITDylib with candidates
      // remaining.
      LLVM_DEBUG(dbgs() << "Phase 1 moving to next JITDylib.\n");
      ++IPLS->CurSearchOrderIndex;
      IPLS->NewJITDylib = true;
    }
  }

  // Remove any weakly referenced candidates that could not be found/generated.
  IPLS->DefGeneratorCandidates.remove_if(
      [](const SymbolStringPtr &Name, SymbolLookupFlags SymLookupFlags) {
        return SymLookupFlags == SymbolLookupFlags::WeaklyReferencedSymbol;
      });

```
- **EN**: Implements logic around `empty`, `size`, `remove_if`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `empty`, `size`, `remove_if` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 2442-2472
```cpp
  // If we get here then we've finished searching all JITDylibs.
  // If we matched all symbols then move to phase 2, otherwise fail the query
  // with a SymbolsNotFound error.
  if (IPLS->DefGeneratorCandidates.empty()) {
    LLVM_DEBUG(dbgs() << "Phase 1 succeeded.\n");
    IPLS->complete(std::move(IPLS));
  } else {
    LLVM_DEBUG(dbgs() << "Phase 1 failed with unresolved symbols.\n");
    IPLS->fail(make_error<SymbolsNotFound>(
        getSymbolStringPool(), IPLS->DefGeneratorCandidates.getSymbolNames()));
  }
}

void ExecutionSession::OL_completeLookup(
    std::unique_ptr<InProgressLookupState> IPLS,
    std::shared_ptr<AsynchronousSymbolQuery> Q,
    RegisterDependenciesFunction RegisterDependencies) {

  LLVM_DEBUG({
    dbgs() << "Entering OL_completeLookup:\n"
           << "  Lookup kind: " << IPLS->K << "\n"
           << "  Search order: " << IPLS->SearchOrder
           << ", Current index = " << IPLS->CurSearchOrderIndex
           << (IPLS->NewJITDylib ? " (entering new JITDylib)" : "") << "\n"
           << "  Lookup set: " << IPLS->LookupSet << "\n"
           << "  Definition generator candidates: "
           << IPLS->DefGeneratorCandidates << "\n"
           << "  Definition generator non-candidates: "
           << IPLS->DefGeneratorNonCandidates << "\n";
  });

```
- **EN**: Implements logic around `complete`, `fail`, `getSymbolStringPool`, `OL_completeLookup`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `complete`, `fail`, `getSymbolStringPool`, `OL_completeLookup`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作。

### Lines 2473-2492
```cpp
  bool QueryComplete = false;
  DenseMap<JITDylib *, JITDylib::UnmaterializedInfosList> CollectedUMIs;

  auto LodgingErr = runSessionLocked([&]() -> Error {
    for (auto &KV : IPLS->SearchOrder) {
      auto &JD = *KV.first;
      auto JDLookupFlags = KV.second;
      LLVM_DEBUG({
        dbgs() << "Visiting \"" << JD.getName() << "\" (" << JDLookupFlags
               << ") with lookup set " << IPLS->LookupSet << ":\n";
      });

      auto Err = IPLS->LookupSet.forEachWithRemoval(
          [&](const SymbolStringPtr &Name,
              SymbolLookupFlags SymLookupFlags) -> Expected<bool> {
            LLVM_DEBUG({
              dbgs() << "  Attempting to match \"" << Name << "\" ("
                     << SymLookupFlags << ")... ";
            });

```
- **EN**: Implements logic around `runSessionLocked`, `dbgs`, `forEachWithRemoval`; this block coordinates ORC symbol lookup or materialization state; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `runSessionLocked`, `dbgs`, `forEachWithRemoval` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 2493-2524
```cpp
            /// Search for the symbol. If not found then continue without
            /// removal.
            auto SymI = JD.Symbols.find(Name);
            if (SymI == JD.Symbols.end()) {
              LLVM_DEBUG(dbgs() << "skipping: not present\n");
              return false;
            }

            // If this is a non-exported symbol and we're matching exported
            // symbols only then skip this symbol without removal.
            if (!SymI->second.getFlags().isExported() &&
                JDLookupFlags ==
                    JITDylibLookupFlags::MatchExportedSymbolsOnly) {
              LLVM_DEBUG(dbgs() << "skipping: not exported\n");
              return false;
            }

            // If we match against a materialization-side-effects only symbol
            // then make sure it is weakly-referenced. Otherwise bail out with
            // an error.
            // FIXME: Use a "materialization-side-effects-only symbols must be
            // weakly referenced" specific error here to reduce confusion.
            if (SymI->second.getFlags().hasMaterializationSideEffectsOnly() &&
                SymLookupFlags != SymbolLookupFlags::WeaklyReferencedSymbol) {
              LLVM_DEBUG({
                dbgs() << "error: "
                          "required, but symbol is has-side-effects-only\n";
              });
              return make_error<SymbolsNotFound>(getSymbolStringPool(),
                                                 SymbolNameVector({Name}));
            }

```
- **EN**: Implements logic around `find`, `dbgs`, `make_error<SymbolsNotFound>`, `SymbolNameVector`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `dbgs`, `make_error<SymbolsNotFound>`, `SymbolNameVector` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 2525-2548
```cpp
            // If we matched against this symbol but it is in the error state
            // then bail out and treat it as a failure to materialize.
            if (SymI->second.getFlags().hasError()) {
              LLVM_DEBUG(dbgs() << "error: symbol is in error state\n");
              auto FailedSymbolsMap = std::make_shared<SymbolDependenceMap>();
              (*FailedSymbolsMap)[&JD] = {Name};
              return make_error<FailedToMaterialize>(
                  getSymbolStringPool(), std::move(FailedSymbolsMap));
            }

            // Otherwise this is a match.

            // If this symbol is already in the required state then notify the
            // query, remove the symbol and continue.
            if (SymI->second.getState() >= Q->getRequiredState()) {
              LLVM_DEBUG(dbgs()
                         << "matched, symbol already in required state\n");
              Q->notifySymbolMetRequiredState(Name, SymI->second.getSymbol());

              // If this symbol is in anything other than the Ready state then
              // we need to track the dependence.
              if (SymI->second.getState() != SymbolState::Ready)
                Q->addQueryDependence(JD, Name);

```
- **EN**: Implements logic around `make_shared<SymbolDependenceMap>`, `make_error<FailedToMaterialize>`, `getSymbolStringPool`, `notifySymbolMetRequiredState`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_shared<SymbolDependenceMap>`, `make_error<FailedToMaterialize>`, `getSymbolStringPool`, `notifySymbolMetRequiredState`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 2549-2569
```cpp
              return true;
            }

            // Otherwise this symbol does not yet meet the required state. Check
            // whether it has a materializer attached, and if so prepare to run
            // it.
            if (SymI->second.hasMaterializerAttached()) {
              assert(SymI->second.getAddress() == ExecutorAddr() &&
                     "Symbol not resolved but already has address?");
              auto UMII = JD.UnmaterializedInfos.find(Name);
              assert(UMII != JD.UnmaterializedInfos.end() &&
                     "Lazy symbol should have UnmaterializedInfo");

              auto UMI = UMII->second;
              assert(UMI->MU && "Materializer should not be null");
              assert(UMI->RT && "Tracker should not be null");
              LLVM_DEBUG({
                dbgs() << "matched, preparing to dispatch MU@" << UMI->MU.get()
                       << " (" << UMI->MU->getName() << ")\n";
              });

```
- **EN**: Implements logic around `assert`, `find`, `dbgs`, `getName`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `find`, `dbgs`, `getName` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 2570-2594
```cpp
              // Move all symbols associated with this MaterializationUnit into
              // materializing state.
              for (auto &KV : UMI->MU->getSymbols()) {
                auto SymK = JD.Symbols.find(KV.first);
                assert(SymK != JD.Symbols.end() &&
                       "No entry for symbol covered by MaterializationUnit");
                SymK->second.setMaterializerAttached(false);
                SymK->second.setState(SymbolState::Materializing);
                JD.UnmaterializedInfos.erase(KV.first);
              }

              // Add MU to the list of MaterializationUnits to be materialized.
              CollectedUMIs[&JD].push_back(std::move(UMI));
            } else
              LLVM_DEBUG(dbgs() << "matched, registering query");

            // Add the query to the PendingQueries list and continue, deleting
            // the element from the lookup set.
            assert(SymI->second.getState() != SymbolState::NeverSearched &&
                   SymI->second.getState() != SymbolState::Ready &&
                   "By this line the symbol should be materializing");
            auto &MI = JD.MaterializingInfos[Name];
            MI.addQuery(Q);
            Q->addQueryDependence(JD, Name);

```
- **EN**: Implements logic around `find`, `assert`, `setMaterializerAttached`, `setState`, and 5 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `find`, `assert`, `setMaterializerAttached`, `setState`, and 5 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 2595-2627
```cpp
            return true;
          });

      JD.shrinkMaterializationInfoMemory();

      // Handle failure.
      if (Err) {

        LLVM_DEBUG({
          dbgs() << "Lookup failed. Detaching query and replacing MUs.\n";
        });

        // Detach the query.
        Q->detach();

        // Replace the MUs.
        for (auto &KV : CollectedUMIs) {
          auto &JD = *KV.first;
          for (auto &UMI : KV.second)
            for (auto &KV2 : UMI->MU->getSymbols()) {
              assert(!JD.UnmaterializedInfos.count(KV2.first) &&
                     "Unexpected materializer in map");
              auto SymI = JD.Symbols.find(KV2.first);
              assert(SymI != JD.Symbols.end() && "Missing symbol entry");
              assert(SymI->second.getState() == SymbolState::Materializing &&
                     "Can not replace symbol that is not materializing");
              assert(!SymI->second.hasMaterializerAttached() &&
                     "MaterializerAttached flag should not be set");
              SymI->second.setMaterializerAttached(true);
              JD.UnmaterializedInfos[KV2.first] = UMI;
            }
        }

```
- **EN**: Implements logic around `shrinkMaterializationInfoMemory`, `dbgs`, `detach`, `assert`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `shrinkMaterializationInfoMemory`, `dbgs`, `detach`, `assert`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 2628-2647
```cpp
        return Err;
      }
    }

    LLVM_DEBUG(dbgs() << "Stripping unmatched weakly-referenced symbols\n");
    IPLS->LookupSet.forEachWithRemoval(
        [&](const SymbolStringPtr &Name, SymbolLookupFlags SymLookupFlags) {
          if (SymLookupFlags == SymbolLookupFlags::WeaklyReferencedSymbol) {
            Q->dropSymbol(Name);
            return true;
          } else
            return false;
        });

    if (!IPLS->LookupSet.empty()) {
      LLVM_DEBUG(dbgs() << "Failing due to unresolved symbols\n");
      return make_error<SymbolsNotFound>(getSymbolStringPool(),
                                         IPLS->LookupSet.getSymbolNames());
    }

```
- **EN**: Implements logic around `forEachWithRemoval`, `dropSymbol`, `make_error<SymbolsNotFound>`, `getSymbolNames`; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `forEachWithRemoval`, `dropSymbol`, `make_error<SymbolsNotFound>`, `getSymbolNames` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 2648-2677
```cpp
    // Record whether the query completed.
    QueryComplete = Q->isComplete();

    LLVM_DEBUG({
      dbgs() << "Query successfully "
             << (QueryComplete ? "completed" : "lodged") << "\n";
    });

    // Move the collected MUs to the OutstandingMUs list.
    if (!CollectedUMIs.empty()) {
      std::lock_guard<std::recursive_mutex> Lock(OutstandingMUsMutex);

      LLVM_DEBUG(dbgs() << "Adding MUs to dispatch:\n");
      for (auto &KV : CollectedUMIs) {
        LLVM_DEBUG({
          auto &JD = *KV.first;
          dbgs() << "  For " << JD.getName() << ": Adding " << KV.second.size()
                 << " MUs.\n";
        });
        for (auto &UMI : KV.second) {
          auto MR = createMaterializationResponsibility(
              *UMI->RT, std::move(UMI->MU->SymbolFlags),
              std::move(UMI->MU->InitSymbol));
          OutstandingMUs.push_back(
              std::make_pair(std::move(UMI->MU), std::move(MR)));
        }
      }
    } else
      LLVM_DEBUG(dbgs() << "No MUs to dispatch.\n");

```
- **EN**: Implements logic around `isComplete`, `dbgs`, `Lock`, `createMaterializationResponsibility`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `isComplete`, `dbgs`, `Lock`, `createMaterializationResponsibility`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 2678-2698
```cpp
    if (RegisterDependencies && !Q->QueryRegistrations.empty()) {
      LLVM_DEBUG(dbgs() << "Registering dependencies\n");
      RegisterDependencies(Q->QueryRegistrations);
    } else
      LLVM_DEBUG(dbgs() << "No dependencies to register\n");

    return Error::success();
  });

  if (LodgingErr) {
    LLVM_DEBUG(dbgs() << "Failing query\n");
    Q->detach();
    Q->handleFailed(std::move(LodgingErr));
    return;
  }

  if (QueryComplete) {
    LLVM_DEBUG(dbgs() << "Completing query\n");
    Q->handleComplete(*this);
  }

```
- **EN**: Implements logic around `RegisterDependencies`, `success`, `detach`, `handleFailed`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `RegisterDependencies`, `success`, `detach`, `handleFailed`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 2699-2719
```cpp
  dispatchOutstandingMUs();
}

void ExecutionSession::OL_completeLookupFlags(
    std::unique_ptr<InProgressLookupState> IPLS,
    unique_function<void(Expected<SymbolFlagsMap>)> OnComplete) {

  auto Result = runSessionLocked([&]() -> Expected<SymbolFlagsMap> {
    LLVM_DEBUG({
      dbgs() << "Entering OL_completeLookupFlags:\n"
             << "  Lookup kind: " << IPLS->K << "\n"
             << "  Search order: " << IPLS->SearchOrder
             << ", Current index = " << IPLS->CurSearchOrderIndex
             << (IPLS->NewJITDylib ? " (entering new JITDylib)" : "") << "\n"
             << "  Lookup set: " << IPLS->LookupSet << "\n"
             << "  Definition generator candidates: "
             << IPLS->DefGeneratorCandidates << "\n"
             << "  Definition generator non-candidates: "
             << IPLS->DefGeneratorNonCandidates << "\n";
    });

```
- **EN**: Implements logic around `dispatchOutstandingMUs`, `OL_completeLookupFlags`, `unique_function<void`, `runSessionLocked`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `dispatchOutstandingMUs`, `OL_completeLookupFlags`, `unique_function<void`, `runSessionLocked`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 2720-2745
```cpp
    SymbolFlagsMap Result;

    // Attempt to find flags for each symbol.
    for (auto &KV : IPLS->SearchOrder) {
      auto &JD = *KV.first;
      auto JDLookupFlags = KV.second;
      LLVM_DEBUG({
        dbgs() << "Visiting \"" << JD.getName() << "\" (" << JDLookupFlags
               << ") with lookup set " << IPLS->LookupSet << ":\n";
      });

      IPLS->LookupSet.forEachWithRemoval([&](const SymbolStringPtr &Name,
                                             SymbolLookupFlags SymLookupFlags) {
        LLVM_DEBUG({
          dbgs() << "  Attempting to match \"" << Name << "\" ("
                 << SymLookupFlags << ")... ";
        });

        // Search for the symbol. If not found then continue without removing
        // from the lookup set.
        auto SymI = JD.Symbols.find(Name);
        if (SymI == JD.Symbols.end()) {
          LLVM_DEBUG(dbgs() << "skipping: not present\n");
          return false;
        }

```
- **EN**: Implements logic around `dbgs`, `forEachWithRemoval`, `find`; this block coordinates ORC symbol lookup or materialization state; executes or prepares runtime behavior for LLVM IR/JIT code; returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `forEachWithRemoval`, `find` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，执行或准备 LLVM IR/JIT 代码的运行时行为，返回子系统相关的计算结果。

### Lines 2746-2767
```cpp
        // If this is a non-exported symbol then it doesn't match. Skip it.
        if (!SymI->second.getFlags().isExported() &&
            JDLookupFlags == JITDylibLookupFlags::MatchExportedSymbolsOnly) {
          LLVM_DEBUG(dbgs() << "skipping: not exported\n");
          return false;
        }

        LLVM_DEBUG({
          dbgs() << "matched, \"" << Name << "\" -> " << SymI->second.getFlags()
                 << "\n";
        });
        Result[Name] = SymI->second.getFlags();
        return true;
      });
    }

    // Remove any weakly referenced symbols that haven't been resolved.
    IPLS->LookupSet.remove_if(
        [](const SymbolStringPtr &Name, SymbolLookupFlags SymLookupFlags) {
          return SymLookupFlags == SymbolLookupFlags::WeaklyReferencedSymbol;
        });

```
- **EN**: Implements logic around `dbgs`, `getFlags`, `remove_if`; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `getFlags`, `remove_if` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 2768-2790
```cpp
    if (!IPLS->LookupSet.empty()) {
      LLVM_DEBUG(dbgs() << "Failing due to unresolved symbols\n");
      return make_error<SymbolsNotFound>(getSymbolStringPool(),
                                         IPLS->LookupSet.getSymbolNames());
    }

    LLVM_DEBUG(dbgs() << "Succeded, result = " << Result << "\n");
    return Result;
  });

  // Run the callback on the result.
  LLVM_DEBUG(dbgs() << "Sending result to handler.\n");
  OnComplete(std::move(Result));
}

void ExecutionSession::OL_destroyMaterializationResponsibility(
    MaterializationResponsibility &MR) {

  assert(MR.SymbolFlags.empty() &&
         "All symbols should have been explicitly materialized or failed");
  MR.JD.unlinkMaterializationResponsibility(MR);
}

```
- **EN**: Implements logic around `make_error<SymbolsNotFound>`, `getSymbolNames`, `OnComplete`, `OL_destroyMaterializationResponsibility`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<SymbolsNotFound>`, `getSymbolNames`, `OnComplete`, `OL_destroyMaterializationResponsibility`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 2791-2820
```cpp
SymbolNameSet ExecutionSession::OL_getRequestedSymbols(
    const MaterializationResponsibility &MR) {
  return MR.JD.getRequestedSymbols(MR.SymbolFlags);
}

Error ExecutionSession::OL_notifyResolved(MaterializationResponsibility &MR,
                                          const SymbolMap &Symbols) {
  LLVM_DEBUG({
    dbgs() << "In " << MR.JD.getName() << " resolving " << Symbols << "\n";
  });
#ifndef NDEBUG
  for (auto &KV : Symbols) {
    auto I = MR.SymbolFlags.find(KV.first);
    assert(I != MR.SymbolFlags.end() &&
           "Resolving symbol outside this responsibility set");
    assert(!I->second.hasMaterializationSideEffectsOnly() &&
           "Can't resolve materialization-side-effects-only symbol");
    if (I->second & JITSymbolFlags::Common) {
      auto WeakOrCommon = JITSymbolFlags::Weak | JITSymbolFlags::Common;
      assert((KV.second.getFlags() & WeakOrCommon) &&
             "Common symbols must be resolved as common or weak");
      assert((KV.second.getFlags() & ~WeakOrCommon) ==
                 (I->second & ~JITSymbolFlags::Common) &&
             "Resolving symbol with incorrect flags");
    } else
      assert(KV.second.getFlags() == I->second &&
             "Resolving symbol with incorrect flags");
  }
#endif

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 2821-2841
```cpp
  return MR.JD.resolve(MR, Symbols);
}

WaitingOnGraph::ExternalState
ExecutionSession::IL_getSymbolState(JITDylib *JD,
                                    NonOwningSymbolStringPtr Name) {
  if (JD->State != JITDylib::Open)
    return WaitingOnGraph::ExternalState::Failed;

  auto I = JD->Symbols.find_as(Name);

  // FIXME: Can we eliminate this possibility if we support query binding?
  if (I == JD->Symbols.end())
    return WaitingOnGraph::ExternalState::Failed;

  if (I->second.getFlags().hasError())
    return WaitingOnGraph::ExternalState::Failed;

  if (I->second.getState() == SymbolState::Ready)
    return WaitingOnGraph::ExternalState::Ready;

```
- **EN**: Implements logic around `resolve`, `IL_getSymbolState`, `find_as`; this block coordinates ORC symbol lookup or materialization state; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `resolve`, `IL_getSymbolState`, `find_as` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 2842-2865
```cpp
  return WaitingOnGraph::ExternalState::None;
}

template <typename UpdateSymbolFn, typename UpdateQueryFn>
void ExecutionSession::IL_collectQueries(
    JITDylib::AsynchronousSymbolQuerySet &Qs,
    WaitingOnGraph::ContainerElementsMap &QualifiedSymbols,
    UpdateSymbolFn &&UpdateSymbol, UpdateQueryFn &&UpdateQuery) {

  for (auto &[JD, Symbols] : QualifiedSymbols) {
    // IL_emit and JITDylib removal are synchronized by the session lock.
    // Since JITDylib removal removes any contained nodes from the
    // WaitingOnGraph, we should be able to assert that all nodes in the
    // WaitingOnGraph have not been removed.
    assert(JD->State == JITDylib::Open &&
           "WaitingOnGraph includes definition in defunct JITDylib");
    for (auto &Symbol : Symbols) {
      // Update symbol table.
      auto I = JD->Symbols.find_as(Symbol);
      assert(I != JD->Symbols.end() &&
             "Failed Symbol missing from JD symbol table");
      auto &Entry = I->second;
      UpdateSymbol(Entry);

```
- **EN**: Implements logic around `IL_collectQueries`, `assert`, `find_as`, `UpdateSymbol`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `IL_collectQueries`, `assert`, `find_as`, `UpdateSymbol` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 2866-2885
```cpp
      // Collect queries.
      auto J = JD->MaterializingInfos.find_as(Symbol);
      if (J != JD->MaterializingInfos.end()) {
        for (auto &Q : J->second.takeAllPendingQueries()) {
          UpdateQuery(*Q, *JD, Symbol, Entry);
          Qs.insert(std::move(Q));
        }
        JD->MaterializingInfos.erase(J);
      }
    }
  }
}

Expected<ExecutionSession::EmitQueries>
ExecutionSession::IL_emit(MaterializationResponsibility &MR,
                          WaitingOnGraph::SimplifyResult SR) {

  if (MR.RT->isDefunct())
    return make_error<ResourceTrackerDefunct>(MR.RT);

```
- **EN**: Implements logic around `find_as`, `UpdateQuery`, `insert`, `erase`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `find_as`, `UpdateQuery`, `insert`, `erase`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 2886-2917
```cpp
  auto &TargetJD = MR.getTargetJITDylib();
  if (TargetJD.State != JITDylib::Open)
    return make_error<StringError>("JITDylib " + TargetJD.getName() +
                                       " is defunct",
                                   inconvertibleErrorCode());

#ifdef EXPENSIVE_CHECKS
  verifySessionState("entering ExecutionSession::IL_emit");
#endif

  auto ER = G.emit(std::move(SR),
                   [this](JITDylib *JD, NonOwningSymbolStringPtr Name) {
                     return IL_getSymbolState(JD, Name);
                   });

  EmitQueries EQ;

  // Handle failed queries.
  for (auto &SN : ER.Failed)
    IL_collectQueries(
        EQ.Failed, SN->defs(),
        [](JITDylib::SymbolTableEntry &E) {
          E.setFlags(E.getFlags() = JITSymbolFlags::HasError);
        },
        [&](AsynchronousSymbolQuery &Q, JITDylib &JD,
            NonOwningSymbolStringPtr Name, JITDylib::SymbolTableEntry &E) {
          auto &FS = EQ.FailedSymsForQuery[&Q];
          if (!FS)
            FS = std::make_shared<SymbolDependenceMap>();
          (*FS)[&JD].insert(SymbolStringPtr(Name));
        });

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 2918-2939
```cpp
  for (auto &FQ : EQ.Failed)
    FQ->detach();

  for (auto &SN : ER.Ready)
    IL_collectQueries(
        EQ.Completed, SN->defs(),
        [](JITDylib::SymbolTableEntry &E) { E.setState(SymbolState::Ready); },
        [](AsynchronousSymbolQuery &Q, JITDylib &JD,
           NonOwningSymbolStringPtr Name, JITDylib::SymbolTableEntry &E) {
          Q.notifySymbolMetRequiredState(SymbolStringPtr(Name), E.getSymbol());
        });

  // std::erase_if is not available in C++17, and llvm::erase_if does not work
  // here.
  for (auto it = EQ.Completed.begin(), end = EQ.Completed.end(); it != end;) {
    if ((*it)->isComplete()) {
      ++it;
    } else {
      it = EQ.Completed.erase(it);
    }
  }

```
- **EN**: Implements logic around `detach`, `IL_collectQueries`, `defs`, `setState`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `detach`, `IL_collectQueries`, `defs`, `setState`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 2940-2961
```cpp
#ifdef EXPENSIVE_CHECKS
  verifySessionState("exiting ExecutionSession::IL_emit");
#endif

  return std::move(EQ);
}

Error ExecutionSession::OL_notifyEmitted(
    MaterializationResponsibility &MR,
    ArrayRef<SymbolDependenceGroup> DepGroups) {
  LLVM_DEBUG({
    dbgs() << "In " << MR.JD.getName() << " emitting " << MR.SymbolFlags
           << "\n";
    if (!DepGroups.empty()) {
      dbgs() << "  Initial dependencies:\n";
      for (auto &SDG : DepGroups) {
        dbgs() << "    Symbols: " << SDG.Symbols
               << ", Dependencies: " << SDG.Dependencies << "\n";
      }
    }
  });

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 2962-3001
```cpp
#ifndef NDEBUG
  SymbolNameSet Visited;
  for (auto &DG : DepGroups) {
    for (auto &Sym : DG.Symbols) {
      assert(MR.SymbolFlags.count(Sym) &&
             "DG contains dependence for symbol outside this MR");
      assert(Visited.insert(Sym).second &&
             "DG contains duplicate entries for Name");
    }
  }
#endif // NDEBUG

  std::vector<std::unique_ptr<WaitingOnGraph::SuperNode>> SNs;
  WaitingOnGraph::ContainerElementsMap Residual;
  {
    auto &JDResidual = Residual[&MR.getTargetJITDylib()];
    for (auto &[Name, Flags] : MR.getSymbols())
      JDResidual.insert(NonOwningSymbolStringPtr(Name));

    for (auto &SDG : DepGroups) {
      WaitingOnGraph::ContainerElementsMap Defs;
      assert(!SDG.Symbols.empty());
      auto &JDDefs = Defs[&MR.getTargetJITDylib()];
      for (auto &Def : SDG.Symbols) {
        JDDefs.insert(NonOwningSymbolStringPtr(Def));
        JDResidual.erase(NonOwningSymbolStringPtr(Def));
      }
      WaitingOnGraph::ContainerElementsMap Deps;
      if (!SDG.Dependencies.empty()) {
        for (auto &[JD, Syms] : SDG.Dependencies) {
          auto &JDDeps = Deps[JD];
          for (auto &Dep : Syms)
            JDDeps.insert(NonOwningSymbolStringPtr(Dep));
        }
      }
      SNs.push_back(std::make_unique<WaitingOnGraph::SuperNode>(
          std::move(Defs), std::move(Deps)));
    }
    if (!JDResidual.empty())
      SNs.push_back(std::make_unique<WaitingOnGraph::SuperNode>(
```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 3002-3029
```cpp
          std::move(Residual), WaitingOnGraph::ContainerElementsMap()));
  }

  auto SR = WaitingOnGraph::simplify(std::move(SNs), GOpRecorder);

  LLVM_DEBUG({
    dbgs() << "  Simplified dependencies:\n";
    for (auto &SN : SR.superNodes()) {

      auto SortedLibs = [](WaitingOnGraph::ContainerElementsMap &C) {
        std::vector<JITDylib *> JDs;
        for (auto &[JD, _] : C)
          JDs.push_back(JD);
        llvm::sort(JDs, [](const JITDylib *LHS, const JITDylib *RHS) {
          return LHS->getName() < RHS->getName();
        });
        return JDs;
      };

      auto SortedNames = [](WaitingOnGraph::ElementSet &Elems) {
        std::vector<NonOwningSymbolStringPtr> Names(Elems.begin(), Elems.end());
        llvm::sort(Names, [](const NonOwningSymbolStringPtr &LHS,
                             const NonOwningSymbolStringPtr &RHS) {
          return *LHS < *RHS;
        });
        return Names;
      };

```
- **EN**: Implements logic around `move`, `simplify`, `dbgs`, `push_back`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `move`, `simplify`, `dbgs`, `push_back`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 3030-3049
```cpp
      dbgs() << "    Defs: {";
      for (auto *JD : SortedLibs(SN->defs())) {
        dbgs() << " (" << JD->getName() << ", [";
        for (auto &Sym : SortedNames(SN->defs()[JD]))
          dbgs() << " " << Sym;
        dbgs() << " ])";
      }
      dbgs() << " }, Deps: {";
      for (auto *JD : SortedLibs(SN->deps())) {
        dbgs() << " (" << JD->getName() << ", [";
        for (auto &Sym : SortedNames(SN->deps()[JD]))
          dbgs() << " " << Sym;
        dbgs() << " ])";
      }
      dbgs() << " }\n";
    }
  });
  auto EmitQueries =
      runSessionLocked([&]() { return IL_emit(MR, std::move(SR)); });

```
- **EN**: Implements logic around `dbgs`, `runSessionLocked`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `runSessionLocked` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 3050-3074
```cpp
  // On error bail out.
  if (!EmitQueries)
    return EmitQueries.takeError();

  // Otherwise notify failed queries, and any updated queries that have been
  // completed.

  // FIXME: Get rid of error return from notifyEmitted.
  SymbolDependenceMap BadDeps;
  {
    for (auto &FQ : EmitQueries->Failed) {
      FQ->detach();
      assert(EmitQueries->FailedSymsForQuery.count(FQ.get()) &&
             "Missing failed symbols for query");
      auto FailedSyms = std::move(EmitQueries->FailedSymsForQuery[FQ.get()]);
      for (auto &[JD, Syms] : *FailedSyms) {
        auto &BadDepsForJD = BadDeps[JD];
        for (auto &Sym : Syms)
          BadDepsForJD.insert(Sym);
      }
      FQ->handleFailed(make_error<FailedToMaterialize>(getSymbolStringPool(),
                                                       std::move(FailedSyms)));
    }
  }

```
- **EN**: Implements logic around `takeError`, `detach`, `assert`, `move`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `detach`, `assert`, `move`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 3075-3096
```cpp
  for (auto &UQ : EmitQueries->Completed)
    UQ->handleComplete(*this);

  // If there are any bad dependencies then return an error.
  if (!BadDeps.empty()) {
    SymbolNameSet BadNames;
    // Note: The name set calculated here is bogus: it includes all symbols in
    //       the MR, not just the ones that failed. We want to remove the error
    //       return path from notifyEmitted anyway, so this is just a brief
    //       placeholder to maintain (roughly) the current error behavior.
    for (auto &[Name, Flags] : MR.getSymbols())
      BadNames.insert(Name);
    MR.SymbolFlags.clear();
    return make_error<UnsatisfiedSymbolDependencies>(
        getSymbolStringPool(), &MR.getTargetJITDylib(), std::move(BadNames),
        std::move(BadDeps), "dependencies removed or in error state");
  }

  MR.SymbolFlags.clear();
  return Error::success();
}

```
- **EN**: Implements logic around `handleComplete`, `insert`, `clear`, `make_error<UnsatisfiedSymbolDependencies>`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `handleComplete`, `insert`, `clear`, `make_error<UnsatisfiedSymbolDependencies>`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 3097-3118
```cpp
Error ExecutionSession::OL_defineMaterializing(
    MaterializationResponsibility &MR, SymbolFlagsMap NewSymbolFlags) {

  LLVM_DEBUG({
    dbgs() << "In " << MR.JD.getName() << " defining materializing symbols "
           << NewSymbolFlags << "\n";
  });
  if (auto AcceptedDefs =
          MR.JD.defineMaterializing(MR, std::move(NewSymbolFlags))) {
    // Add all newly accepted symbols to this responsibility object.
    for (auto &KV : *AcceptedDefs)
      MR.SymbolFlags.insert(KV);
    return Error::success();
  } else
    return AcceptedDefs.takeError();
}

std::pair<JITDylib::AsynchronousSymbolQuerySet,
          std::shared_ptr<SymbolDependenceMap>>
ExecutionSession::IL_failSymbols(JITDylib &JD,
                                 const SymbolNameVector &SymbolsToFail) {

```
- **EN**: Implements logic around `OL_defineMaterializing`, `dbgs`, `defineMaterializing`, `insert`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `OL_defineMaterializing`, `dbgs`, `defineMaterializing`, `insert`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 3119-3139
```cpp
#ifdef EXPENSIVE_CHECKS
  verifySessionState("entering ExecutionSession::IL_failSymbols");
#endif

  // Early out in the easy case.
  if (SymbolsToFail.empty())
    return {};

  JITDylib::AsynchronousSymbolQuerySet FailedQueries;
  auto Fail = [&](JITDylib *FailJD, NonOwningSymbolStringPtr FailSym) {
    auto I = FailJD->Symbols.find_as(FailSym);
    assert(I != FailJD->Symbols.end());
    I->second.setFlags(I->second.getFlags() | JITSymbolFlags::HasError);
    auto J = FailJD->MaterializingInfos.find_as(FailSym);
    if (J != FailJD->MaterializingInfos.end()) {
      for (auto &Q : J->second.takeAllPendingQueries())
        FailedQueries.insert(std::move(Q));
      FailJD->MaterializingInfos.erase(J);
    }
  };

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 3140-3166
```cpp
  auto FailedSymbolsMap = std::make_shared<SymbolDependenceMap>();

  {
    auto &FailedSymsForJD = (*FailedSymbolsMap)[&JD];
    for (auto &Sym : SymbolsToFail) {
      FailedSymsForJD.insert(Sym);
      Fail(&JD, NonOwningSymbolStringPtr(Sym));
    }
  }

  WaitingOnGraph::ContainerElementsMap ToFail;
  auto &JDToFail = ToFail[&JD];
  for (auto &Sym : SymbolsToFail)
    JDToFail.insert(NonOwningSymbolStringPtr(Sym));

  auto FailedSNs = G.fail(ToFail, GOpRecorder);

  for (auto &SN : FailedSNs) {
    for (auto &[FailJD, Defs] : SN->defs()) {
      auto &FailedSymsForFailJD = (*FailedSymbolsMap)[FailJD];
      for (auto &Def : Defs) {
        FailedSymsForFailJD.insert(SymbolStringPtr(Def));
        Fail(FailJD, Def);
      }
    }
  }

```
- **EN**: Implements logic around `make_shared<SymbolDependenceMap>`, `insert`, `Fail`, `fail`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `make_shared<SymbolDependenceMap>`, `insert`, `Fail`, `fail` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 3167-3187
```cpp
  // Detach all failed queries.
  for (auto &Q : FailedQueries)
    Q->detach();

#ifdef EXPENSIVE_CHECKS
  verifySessionState("exiting ExecutionSession::IL_failSymbols");
#endif

  return std::make_pair(std::move(FailedQueries), std::move(FailedSymbolsMap));
}

void ExecutionSession::OL_notifyFailed(MaterializationResponsibility &MR) {

  LLVM_DEBUG({
    dbgs() << "In " << MR.JD.getName() << " failing materialization for "
           << MR.SymbolFlags << "\n";
  });

  if (MR.SymbolFlags.empty())
    return;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 3188-3210
```cpp
  SymbolNameVector SymbolsToFail;
  for (auto &[Name, Flags] : MR.SymbolFlags)
    SymbolsToFail.push_back(Name);
  MR.SymbolFlags.clear();

  JITDylib::AsynchronousSymbolQuerySet FailedQueries;
  std::shared_ptr<SymbolDependenceMap> FailedSymbols;

  std::tie(FailedQueries, FailedSymbols) = runSessionLocked([&]() {
    // If the tracker is defunct then there's nothing to do here.
    if (MR.RT->isDefunct())
      return std::pair<JITDylib::AsynchronousSymbolQuerySet,
                       std::shared_ptr<SymbolDependenceMap>>();
    return IL_failSymbols(MR.getTargetJITDylib(), SymbolsToFail);
  });

  for (auto &Q : FailedQueries) {
    Q->detach();
    Q->handleFailed(
        make_error<FailedToMaterialize>(getSymbolStringPool(), FailedSymbols));
  }
}

```
- **EN**: Implements logic around `push_back`, `clear`, `tie`, `shared_ptr<SymbolDependenceMap>>`, and 4 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `clear`, `tie`, `shared_ptr<SymbolDependenceMap>>`, and 4 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 3211-3233
```cpp
Error ExecutionSession::OL_replace(MaterializationResponsibility &MR,
                                   std::unique_ptr<MaterializationUnit> MU) {
  for (auto &KV : MU->getSymbols()) {
    assert(MR.SymbolFlags.count(KV.first) &&
           "Replacing definition outside this responsibility set");
    MR.SymbolFlags.erase(KV.first);
  }

  if (MU->getInitializerSymbol() == MR.InitSymbol)
    MR.InitSymbol = nullptr;

  LLVM_DEBUG(MR.JD.getExecutionSession().runSessionLocked([&]() {
    dbgs() << "In " << MR.JD.getName() << " replacing symbols with " << *MU
           << "\n";
  }););

  return MR.JD.replace(MR, std::move(MU));
}

Expected<std::unique_ptr<MaterializationResponsibility>>
ExecutionSession::OL_delegate(MaterializationResponsibility &MR,
                              const SymbolNameSet &Symbols) {

```
- **EN**: Implements logic around `OL_replace`, `assert`, `erase`, `dbgs`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `OL_replace`, `assert`, `erase`, `dbgs`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 3234-3253
```cpp
  SymbolStringPtr DelegatedInitSymbol;
  SymbolFlagsMap DelegatedFlags;

  for (auto &Name : Symbols) {
    auto I = MR.SymbolFlags.find(Name);
    assert(I != MR.SymbolFlags.end() &&
           "Symbol is not tracked by this MaterializationResponsibility "
           "instance");

    DelegatedFlags[Name] = std::move(I->second);
    if (Name == MR.InitSymbol)
      std::swap(MR.InitSymbol, DelegatedInitSymbol);

    MR.SymbolFlags.erase(I);
  }

  return MR.JD.delegate(MR, std::move(DelegatedFlags),
                        std::move(DelegatedInitSymbol));
}

```
- **EN**: Implements logic around `find`, `assert`, `move`, `swap`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `assert`, `move`, `swap`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 3254-3265
```cpp
#ifndef NDEBUG
void ExecutionSession::dumpDispatchInfo(Task &T) {
  runSessionLocked([&]() {
    dbgs() << "Dispatching: ";
    T.printDescription(dbgs());
    dbgs() << "\n";
  });
}
#endif // NDEBUG

} // End namespace orc.
} // End namespace llvm.
```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

## Key Concepts / 关键概念

- **ORC JIT / ORC JIT**:
  - **EN**: Coordinates JITDylibs, symbol materialization, execution sessions, and asynchronous compilation flows
  - **CN**: 协调 JITDylib、符号物化、ExecutionSession 与异步编译流程
- **Symbol materialization / 符号物化**:
  - **EN**: Defers code or data generation until symbols are requested
  - **CN**: 将代码或数据的生成延迟到符号真正被请求时
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/Core.h`, `llvm/ADT/STLExtras.h`, `llvm/Config/llvm-config.h`, `llvm/ExecutionEngine/Orc/AbsoluteSymbols.h`, `llvm/ExecutionEngine/Orc/DebugUtils.h`, `llvm/ExecutionEngine/Orc/Shared/OrcError.h`, `llvm/Support/FormatVariadic.h`, `llvm/Support/MSVCErrorWorkarounds.h`, `llvm/Support/raw_ostream.h`, `condition_variable`, `future`, `optional`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Support
