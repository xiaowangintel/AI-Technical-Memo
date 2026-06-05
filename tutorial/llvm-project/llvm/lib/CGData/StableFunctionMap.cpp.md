# StableFunctionMap.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/CGData/StableFunctionMap.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This implements the functionality for the StableFunctionMap class, which manages the mapping of stable function hashes to their metadata. It includes methods for inserting, merging, and finalizing function entries, as well as utilities for handling function names and IDs.
  - **CN**: 实现代码生成剖析数据的数据结构、读取器与写出器。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- StableFunctionMap.cpp ---------------------------------------------===//
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
//
// This implements the functionality for the StableFunctionMap class, which
// manages the mapping of stable function hashes to their metadata. It includes
// methods for inserting, merging, and finalizing function entries, as well as
// utilities for handling function names and IDs.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 15-21
```cpp

#include "llvm/CGData/StableFunctionMap.h"
#include "llvm/ADT/SmallSet.h"
#include "llvm/CGData/StableFunctionMapRecord.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/CGData/StableFunctionMap.h`, `llvm/ADT/SmallSet.h`, `llvm/CGData/StableFunctionMapRecord.h`, `llvm/Support/CommandLine.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/CGData/StableFunctionMap.h`, `llvm/ADT/SmallSet.h`, `llvm/CGData/StableFunctionMapRecord.h`, `llvm/Support/CommandLine.h`。

### Lines 22-35
```cpp
#define DEBUG_TYPE "stable-function-map"

using namespace llvm;

static cl::opt<unsigned>
    GlobalMergingMinMerges("global-merging-min-merges",
                           cl::desc("Minimum number of similar functions with "
                                    "the same hash required for merging."),
                           cl::init(2), cl::Hidden);
static cl::opt<unsigned> GlobalMergingMinInstrs(
    "global-merging-min-instrs",
    cl::desc("The minimum instruction count required when merging functions."),
    cl::init(1), cl::Hidden);
static cl::opt<unsigned> GlobalMergingMaxParams(
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 36-49
```cpp
    "global-merging-max-params",
    cl::desc(
        "The maximum number of parameters allowed when merging functions."),
    cl::init(std::numeric_limits<unsigned>::max()), cl::Hidden);
static cl::opt<bool> GlobalMergingSkipNoParams(
    "global-merging-skip-no-params",
    cl::desc("Skip merging functions with no parameters."), cl::init(true),
    cl::Hidden);
static cl::opt<double> GlobalMergingInstOverhead(
    "global-merging-inst-overhead",
    cl::desc("The overhead cost associated with each instruction when lowering "
             "to machine instruction."),
    cl::init(1.2), cl::Hidden);
static cl::opt<double> GlobalMergingParamOverhead(
```
- **EN**: Implements logic around `desc`, `init`, `GlobalMergingSkipNoParams`, `GlobalMergingInstOverhead`, and 1 more symbols.
- **CN**: 围绕 `desc`, `init`, `GlobalMergingSkipNoParams`, `GlobalMergingInstOverhead`, and 1 more symbols 实现具体逻辑。

### Lines 50-63
```cpp
    "global-merging-param-overhead",
    cl::desc("The overhead cost associated with each parameter when merging "
             "functions."),
    cl::init(2.0), cl::Hidden);
static cl::opt<double>
    GlobalMergingCallOverhead("global-merging-call-overhead",
                              cl::desc("The overhead cost associated with each "
                                       "function call when merging functions."),
                              cl::init(1.0), cl::Hidden);
static cl::opt<double> GlobalMergingExtraThreshold(
    "global-merging-extra-threshold",
    cl::desc("An additional cost threshold that must be exceeded for merging "
             "to be considered beneficial."),
    cl::init(0.0), cl::Hidden);
```
- **EN**: Implements logic around `desc`, `init`, `GlobalMergingCallOverhead`, `GlobalMergingExtraThreshold`.
- **CN**: 围绕 `desc`, `init`, `GlobalMergingCallOverhead`, `GlobalMergingExtraThreshold` 实现具体逻辑。

### Lines 64-75
```cpp

unsigned StableFunctionMap::getIdOrCreateForName(StringRef Name) {
  auto It = NameToId.find(Name);
  if (It != NameToId.end())
    return It->second;
  unsigned Id = IdToName.size();
  assert(Id == NameToId.size() && "ID collision");
  IdToName.emplace_back(Name.str());
  NameToId[IdToName.back()] = Id;
  return Id;
}

```
- **EN**: Implements logic around `getIdOrCreateForName`, `find`, `end`, `size`, and 3 more symbols.
- **CN**: 围绕 `getIdOrCreateForName`, `find`, `end`, `size`, and 3 more symbols 实现具体逻辑。

### Lines 76-89
```cpp
std::optional<std::string> StableFunctionMap::getNameForId(unsigned Id) const {
  if (Id >= IdToName.size())
    return std::nullopt;
  return IdToName[Id];
}

void StableFunctionMap::insert(const StableFunction &Func) {
  assert(!Finalized && "Cannot insert after finalization");
  auto FuncNameId = getIdOrCreateForName(Func.FunctionName);
  auto ModuleNameId = getIdOrCreateForName(Func.ModuleName);
  auto IndexOperandHashMap = std::make_unique<IndexOperandHashMapType>();
  for (auto &[Index, Hash] : Func.IndexOperandHashes)
    (*IndexOperandHashMap)[Index] = Hash;
  auto FuncEntry = std::make_unique<StableFunctionEntry>(
```
- **EN**: Implements logic around `getNameForId`, `size`, `insert`, `assert`, and 2 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `getNameForId`, `size`, `insert`, `assert`, and 2 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 90-103
```cpp
      Func.Hash, FuncNameId, ModuleNameId, Func.InstCount,
      std::move(IndexOperandHashMap));
  insert(std::move(FuncEntry));
}

void StableFunctionMap::merge(const StableFunctionMap &OtherMap) {
  assert(!Finalized && "Cannot merge after finalization");
  deserializeLazyLoadingEntries();
  for (auto &[Hash, Funcs] : OtherMap.HashToFuncs) {
    auto &ThisFuncs = HashToFuncs[Hash].Entries;
    for (auto &Func : Funcs.Entries) {
      auto FuncNameId =
          getIdOrCreateForName(*OtherMap.getNameForId(Func->FunctionNameId));
      auto ModuleNameId =
```
- **EN**: Implements logic around `move`, `insert`, `merge`, `assert`, and 2 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `move`, `insert`, `merge`, `assert`, and 2 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 104-113
```cpp
          getIdOrCreateForName(*OtherMap.getNameForId(Func->ModuleNameId));
      auto ClonedIndexOperandHashMap =
          std::make_unique<IndexOperandHashMapType>(*Func->IndexOperandHashMap);
      ThisFuncs.emplace_back(std::make_unique<StableFunctionEntry>(
          Func->Hash, FuncNameId, ModuleNameId, Func->InstCount,
          std::move(ClonedIndexOperandHashMap)));
    }
  }
}

```
- **EN**: Implements logic around `getIdOrCreateForName`, `make_unique`, `emplace_back`, `move`; this block works with hashed storage or cache state.
- **CN**: 围绕 `getIdOrCreateForName`, `make_unique`, `emplace_back`, `move` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 114-127
```cpp
size_t StableFunctionMap::size(SizeType Type) const {
  switch (Type) {
  case UniqueHashCount:
    return HashToFuncs.size();
  case TotalFunctionCount: {
    deserializeLazyLoadingEntries();
    size_t Count = 0;
    for (auto &Funcs : HashToFuncs)
      Count += Funcs.second.Entries.size();
    return Count;
  }
  case MergeableFunctionCount: {
    deserializeLazyLoadingEntries();
    size_t Count = 0;
```
- **EN**: Implements logic around `size`, `deserializeLazyLoadingEntries`; this block uses `switch`-style dispatch.
- **CN**: 围绕 `size`, `deserializeLazyLoadingEntries` 实现具体逻辑；该代码块使用 `switch` 风格分派。

### Lines 128-136
```cpp
    for (auto &[Hash, Funcs] : HashToFuncs)
      if (Funcs.Entries.size() >= 2)
        Count += Funcs.Entries.size();
    return Count;
  }
  }
  llvm_unreachable("Unhandled size type");
}

```
- **EN**: Implements logic around `size`, `llvm_unreachable`; this block works with hashed storage or cache state.
- **CN**: 围绕 `size`, `llvm_unreachable` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 137-145
```cpp
const StableFunctionMap::StableFunctionEntries &
StableFunctionMap::at(HashFuncsMapType::key_type FunctionHash) const {
  auto It = HashToFuncs.find(FunctionHash);
  assert(It != HashToFuncs.end() && "FunctionHash not found!");
  if (isLazilyLoaded())
    deserializeLazyLoadingEntry(It);
  return It->second.Entries;
}

```
- **EN**: Implements logic around `at`, `find`, `assert`, `isLazilyLoaded`, and 1 more symbols.
- **CN**: 围绕 `at`, `find`, `assert`, `isLazilyLoaded`, and 1 more symbols 实现具体逻辑。

### Lines 146-158
```cpp
void StableFunctionMap::deserializeLazyLoadingEntry(
    HashFuncsMapType::iterator It) const {
  assert(isLazilyLoaded() && "Cannot deserialize non-lazily-loaded map");
  auto &[Hash, Storage] = *It;
  std::call_once(Storage.LazyLoadFlag,
                 [this, HashArg = Hash, &StorageArg = Storage]() {
                   for (auto Offset : StorageArg.Offsets)
                     StableFunctionMapRecord::deserializeEntry(
                         reinterpret_cast<const unsigned char *>(Offset),
                         HashArg, const_cast<StableFunctionMap *>(this));
                 });
}

```
- **EN**: Implements logic around `deserializeLazyLoadingEntry`, `assert`, `call_once`, `deserializeEntry`; this block works with hashed storage or cache state.
- **CN**: 围绕 `deserializeLazyLoadingEntry`, `assert`, `call_once`, `deserializeEntry` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 159-165
```cpp
void StableFunctionMap::deserializeLazyLoadingEntries() const {
  if (!isLazilyLoaded())
    return;
  for (auto It = HashToFuncs.begin(); It != HashToFuncs.end(); ++It)
    deserializeLazyLoadingEntry(It);
}

```
- **EN**: Implements logic around `deserializeLazyLoadingEntries`, `isLazilyLoaded`, `begin`, `deserializeLazyLoadingEntry`.
- **CN**: 围绕 `deserializeLazyLoadingEntries`, `isLazilyLoaded`, `begin`, `deserializeLazyLoadingEntry` 实现具体逻辑。

### Lines 166-173
```cpp
const StableFunctionMap::HashFuncsMapType &
StableFunctionMap::getFunctionMap() const {
  // Ensure all entries are deserialized before returning the raw map.
  if (isLazilyLoaded())
    deserializeLazyLoadingEntries();
  return HashToFuncs;
}

```
- **EN**: Implements logic around `getFunctionMap`, `isLazilyLoaded`, `deserializeLazyLoadingEntries`.
- **CN**: 围绕 `getFunctionMap`, `isLazilyLoaded`, `deserializeLazyLoadingEntries` 实现具体逻辑。

### Lines 174-187
```cpp
using ParamLocs = SmallVector<IndexPair>;
static void
removeIdenticalIndexPair(StableFunctionMap::StableFunctionEntries &SFS) {
  auto &RSF = SFS[0];
  unsigned StableFunctionCount = SFS.size();

  SmallVector<IndexPair> ToDelete;
  for (auto &[Pair, Hash] : *(RSF->IndexOperandHashMap)) {
    bool Identical = true;
    for (unsigned J = 1; J < StableFunctionCount; ++J) {
      auto &SF = SFS[J];
      const auto &SHash = SF->IndexOperandHashMap->at(Pair);
      if (Hash != SHash) {
        Identical = false;
```
- **EN**: Implements logic around `removeIdenticalIndexPair`, `size`, `at`; this block works with hashed storage or cache state.
- **CN**: 围绕 `removeIdenticalIndexPair`, `size`, `at` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 188-197
```cpp
        break;
      }
    }

    // No need to parameterize them if the hashes are identical across stable
    // functions.
    if (Identical)
      ToDelete.emplace_back(Pair);
  }

```
- **EN**: Implements logic around `emplace_back`.
- **CN**: 围绕 `emplace_back` 实现具体逻辑。

### Lines 198-207
```cpp
  for (auto &Pair : ToDelete)
    for (auto &SF : SFS)
      SF->IndexOperandHashMap->erase(Pair);
}

static bool isProfitable(const StableFunctionMap::StableFunctionEntries &SFS) {
  unsigned StableFunctionCount = SFS.size();
  if (StableFunctionCount < GlobalMergingMinMerges)
    return false;

```
- **EN**: Implements logic around `erase`, `isProfitable`, `size`.
- **CN**: 围绕 `erase`, `isProfitable`, `size` 实现具体逻辑。

### Lines 208-221
```cpp
  unsigned InstCount = SFS[0]->InstCount;
  if (InstCount < GlobalMergingMinInstrs)
    return false;

  double Cost = 0.0;
  SmallSet<stable_hash, 8> UniqueHashVals;
  for (auto &SF : SFS) {
    UniqueHashVals.clear();
    for (auto &[IndexPair, Hash] : *SF->IndexOperandHashMap)
      UniqueHashVals.insert(Hash);
    unsigned ParamCount = UniqueHashVals.size();
    if (ParamCount > GlobalMergingMaxParams)
      return false;
    // Theoretically, if ParamCount is 0, it results in identical code folding
```
- **EN**: Implements logic around `clear`, `insert`, `size`; this block works with hashed storage or cache state.
- **CN**: 围绕 `clear`, `insert`, `size` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 222-231
```cpp
    // (ICF), which we can skip merging here since the linker already handles
    // ICF. This pass would otherwise introduce unnecessary thunks that are
    // merely direct jumps. However, enabling this could be beneficial depending
    // on downstream passes, so we provide an option for it.
    if (GlobalMergingSkipNoParams && ParamCount == 0)
      return false;
    Cost += ParamCount * GlobalMergingParamOverhead + GlobalMergingCallOverhead;
  }
  Cost += GlobalMergingExtraThreshold;

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 232-242
```cpp
  double Benefit =
      InstCount * (StableFunctionCount - 1) * GlobalMergingInstOverhead;
  bool Result = Benefit > Cost;
  LLVM_DEBUG(dbgs() << "isProfitable: Hash = " << SFS[0]->Hash << ", "
                    << "StableFunctionCount = " << StableFunctionCount
                    << ", InstCount = " << InstCount
                    << ", Benefit = " << Benefit << ", Cost = " << Cost
                    << ", Result = " << (Result ? "true" : "false") << "\n");
  return Result;
}

```
- **EN**: Implements logic around `dbgs`; this block works with hashed storage or cache state.
- **CN**: 围绕 `dbgs` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 243-249
```cpp
void StableFunctionMap::finalize(bool SkipTrim) {
  deserializeLazyLoadingEntries();
  SmallVector<HashFuncsMapType::iterator> ToDelete;
  for (auto It = HashToFuncs.begin(); It != HashToFuncs.end(); ++It) {
    auto &[StableHash, Storage] = *It;
    auto &SFS = Storage.Entries;

```
- **EN**: Implements logic around `finalize`, `deserializeLazyLoadingEntries`, `begin`.
- **CN**: 围绕 `finalize`, `deserializeLazyLoadingEntries`, `begin` 实现具体逻辑。

### Lines 250-258
```cpp
    // Group stable functions by ModuleIdentifier.
    llvm::stable_sort(SFS, [&](const std::unique_ptr<StableFunctionEntry> &L,
                               const std::unique_ptr<StableFunctionEntry> &R) {
      return *getNameForId(L->ModuleNameId) < *getNameForId(R->ModuleNameId);
    });

    // Consider the first function as the root function.
    auto &RSF = SFS[0];

```
- **EN**: Implements logic around `stable_sort`, `getNameForId`.
- **CN**: 围绕 `stable_sort`, `getNameForId` 实现具体逻辑。

### Lines 259-272
```cpp
    bool Invalid = false;
    unsigned StableFunctionCount = SFS.size();
    for (unsigned I = 1; I < StableFunctionCount; ++I) {
      auto &SF = SFS[I];
      assert(RSF->Hash == SF->Hash);
      if (RSF->InstCount != SF->InstCount) {
        Invalid = true;
        break;
      }
      if (RSF->IndexOperandHashMap->size() != SF->IndexOperandHashMap->size()) {
        Invalid = true;
        break;
      }
      for (auto &P : *RSF->IndexOperandHashMap) {
```
- **EN**: Implements logic around `size`, `assert`; this block works with hashed storage or cache state.
- **CN**: 围绕 `size`, `assert` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 273-284
```cpp
        auto &InstOpndIndex = P.first;
        if (!SF->IndexOperandHashMap->count(InstOpndIndex)) {
          Invalid = true;
          break;
        }
      }
    }
    if (Invalid) {
      ToDelete.push_back(It);
      continue;
    }

```
- **EN**: Implements logic around `count`, `push_back`.
- **CN**: 围绕 `count`, `push_back` 实现具体逻辑。

### Lines 285-291
```cpp
    if (SkipTrim)
      continue;

    // Trim the index pair that has the same operand hash across
    // stable functions.
    removeIdenticalIndexPair(SFS);

```
- **EN**: Implements logic around `removeIdenticalIndexPair`; this block works with hashed storage or cache state.
- **CN**: 围绕 `removeIdenticalIndexPair` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 292-299
```cpp
    if (!isProfitable(SFS))
      ToDelete.push_back(It);
  }
  for (auto It : ToDelete)
    HashToFuncs.erase(It);

  Finalized = true;
}
```
- **EN**: Implements logic around `isProfitable`, `push_back`, `erase`.
- **CN**: 围绕 `isProfitable`, `push_back`, `erase` 实现具体逻辑。

## Key Concepts / 关键概念

- **CodeGen profiling data / 代码生成剖析数据**:
  - **EN**: Stores or transfers profile-like data consumed by code-generation workflows.
  - **CN**: 存储或传输代码生成流程消费的剖析类数据。
- **LLVM container usage / LLVM 容器使用**:
  - **EN**: Relies on LLVM ADT containers for performance-conscious in-memory data management.
  - **CN**: 依赖 LLVM ADT 容器来进行注重性能的内存数据管理。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/CGData/StableFunctionMap.h`, `llvm/ADT/SmallSet.h`, `llvm/CGData/StableFunctionMapRecord.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`
- **Subsystem categories / 子系统类别**: support-library helpers / Support 库辅助功能 (2), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
