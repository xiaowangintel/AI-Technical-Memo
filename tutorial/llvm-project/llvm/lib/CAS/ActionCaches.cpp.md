# ActionCaches.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/CAS/ActionCaches.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: \file This file implements the underlying ActionCache implementations.
  - **CN**: 实现内容寻址存储、磁盘缓存以及相关 schema 支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-21
```cpp
///
/// \file This file implements the underlying ActionCache implementations.
///
//===----------------------------------------------------------------------===//

#include "BuiltinCAS.h"
#include "llvm/ADT/TrieRawHashMap.h"
#include "llvm/CAS/ActionCache.h"
#include "llvm/CAS/OnDiskCASLogger.h"
#include "llvm/CAS/OnDiskKeyValueDB.h"
#include "llvm/CAS/UnifiedOnDiskCache.h"
#include "llvm/Config/llvm-config.h"
#include "llvm/Support/BLAKE3.h"
#include "llvm/Support/Errc.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `BuiltinCAS.h`, `llvm/ADT/TrieRawHashMap.h`, `llvm/CAS/ActionCache.h`, `llvm/CAS/OnDiskCASLogger.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `BuiltinCAS.h`, `llvm/ADT/TrieRawHashMap.h`, `llvm/CAS/ActionCache.h`, `llvm/CAS/OnDiskCASLogger.h`。

### Lines 22-28
```cpp

#define DEBUG_TYPE "cas-action-caches"

using namespace llvm;
using namespace llvm::cas;

namespace {
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 29-39
```cpp

using HasherT = BLAKE3;
using HashType = decltype(HasherT::hash(std::declval<ArrayRef<uint8_t> &>()));

template <size_t Size> class CacheEntry {
public:
  CacheEntry() = default;
  CacheEntry(ArrayRef<uint8_t> Hash) { llvm::copy(Hash, Value.data()); }
  CacheEntry(const CacheEntry &Entry) { llvm::copy(Entry.Value, Value.data()); }
  ArrayRef<uint8_t> getValue() const { return Value; }

```
- **EN**: Introduces declarations for `CacheEntry`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `CacheEntry` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 40-46
```cpp
private:
  std::array<uint8_t, Size> Value;
};

/// Builtin InMemory ActionCache that stores the mapping in memory.
class InMemoryActionCache final : public ActionCache {
public:
```
- **EN**: Introduces declarations for `InMemoryActionCache`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `InMemoryActionCache` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 47-54
```cpp
  InMemoryActionCache()
      : ActionCache(builtin::BuiltinCASContext::getDefaultContext()) {}

  Error putImpl(ArrayRef<uint8_t> ActionKey, const CASID &Result,
                bool CanBeDistributed) final;
  Expected<std::optional<CASID>> getImpl(ArrayRef<uint8_t> ActionKey,
                                         bool CanBeDistributed) const final;

```
- **EN**: Implements logic around `InMemoryActionCache`, `ActionCache`, `putImpl`, `getImpl`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `InMemoryActionCache`, `ActionCache`, `putImpl`, `getImpl` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 55-62
```cpp
  Error validate() const final {
    return createStringError("InMemoryActionCache doesn't support validate()");
  }

private:
  using DataT = CacheEntry<sizeof(HashType)>;
  using InMemoryCacheT = ThreadSafeTrieRawHashMap<DataT, sizeof(HashType)>;

```
- **EN**: Implements logic around `validate`, `createStringError`, `CacheEntry`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `validate`, `createStringError`, `CacheEntry` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 63-73
```cpp
  InMemoryCacheT Cache;
};

/// Builtin basic OnDiskActionCache that uses one underlying OnDiskKeyValueDB.
class OnDiskActionCache final : public ActionCache {
public:
  Error putImpl(ArrayRef<uint8_t> ActionKey, const CASID &Result,
                bool CanBeDistributed) final;
  Expected<std::optional<CASID>> getImpl(ArrayRef<uint8_t> ActionKey,
                                         bool CanBeDistributed) const final;

```
- **EN**: Introduces declarations for `OnDiskActionCache`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `OnDiskActionCache` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 74-80
```cpp
  static Expected<std::unique_ptr<OnDiskActionCache>> create(StringRef Path);

  Error validate() const final;

private:
  static StringRef getHashName() { return "BLAKE3"; }

```
- **EN**: Implements logic around `create`, `validate`, `getHashName`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `create`, `validate`, `getHashName` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 81-87
```cpp
  OnDiskActionCache(std::unique_ptr<ondisk::OnDiskKeyValueDB> DB);

  std::unique_ptr<ondisk::OnDiskKeyValueDB> DB;
  using DataT = CacheEntry<sizeof(HashType)>;
};

/// Builtin unified ActionCache that wraps around UnifiedOnDiskCache to provide
```
- **EN**: Implements logic around `OnDiskActionCache`, `CacheEntry`; this block works with hashed storage or cache state.
- **CN**: 围绕 `OnDiskActionCache`, `CacheEntry` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 88-95
```cpp
/// access to its ActionCache.
class UnifiedOnDiskActionCache final : public ActionCache {
public:
  Error putImpl(ArrayRef<uint8_t> ActionKey, const CASID &Result,
                bool CanBeDistributed) final;
  Expected<std::optional<CASID>> getImpl(ArrayRef<uint8_t> ActionKey,
                                         bool CanBeDistributed) const final;

```
- **EN**: Introduces declarations for `UnifiedOnDiskActionCache`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `UnifiedOnDiskActionCache` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 96-104
```cpp
  UnifiedOnDiskActionCache(std::shared_ptr<ondisk::UnifiedOnDiskCache> UniDB);

  Error validate() const final;

private:
  std::shared_ptr<ondisk::UnifiedOnDiskCache> UniDB;
};
} // end namespace

```
- **EN**: Implements logic around `UnifiedOnDiskActionCache`, `validate`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `UnifiedOnDiskActionCache`, `validate` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 105-118
```cpp
static Error createResultCachePoisonedError(ArrayRef<uint8_t> KeyHash,
                                            const CASContext &Context,
                                            CASID Output,
                                            ArrayRef<uint8_t> ExistingOutput) {
  std::string Existing =
      CASID::create(&Context, toStringRef(ExistingOutput)).toString();
  SmallString<64> Key;
  toHex(KeyHash, /*LowerCase=*/true, Key);
  return createStringError(std::make_error_code(std::errc::invalid_argument),
                           "cache poisoned for '" + Key + "' (new='" +
                               Output.toString() + "' vs. existing '" +
                               Existing + "')");
}

```
- **EN**: Implements logic around `createResultCachePoisonedError`, `create`, `toHex`, `createStringError`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `createResultCachePoisonedError`, `create`, `toHex`, `createStringError`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 119-127
```cpp
Expected<std::optional<CASID>>
InMemoryActionCache::getImpl(ArrayRef<uint8_t> Key,
                             bool /*CanBeDistributed*/) const {
  auto Result = Cache.find(Key);
  if (!Result)
    return std::nullopt;
  return CASID::create(&getContext(), toStringRef(Result->Data.getValue()));
}

```
- **EN**: Implements logic around `getImpl`, `find`, `create`; this block works with hashed storage or cache state.
- **CN**: 围绕 `getImpl`, `find`, `create` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 128-137
```cpp
Error InMemoryActionCache::putImpl(ArrayRef<uint8_t> Key, const CASID &Result,
                                   bool /*CanBeDistributed*/) {
  DataT Expected(Result.getHash());
  const InMemoryCacheT::value_type &Cached = *Cache.insertLazy(
      Key, [&](auto ValueConstructor) { ValueConstructor.emplace(Expected); });

  const DataT &Observed = Cached.Data;
  if (Expected.getValue() == Observed.getValue())
    return Error::success();

```
- **EN**: Implements logic around `putImpl`, `Expected`, `insertLazy`, `emplace`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `putImpl`, `Expected`, `insertLazy`, `emplace`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 138-147
```cpp
  return createResultCachePoisonedError(Key, getContext(), Result,
                                        Observed.getValue());
}

namespace llvm::cas {

std::unique_ptr<ActionCache> createInMemoryActionCache() {
  return std::make_unique<InMemoryActionCache>();
}

```
- **EN**: Introduces declarations for `llvm::cas`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `llvm::cas` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 148-154
```cpp
} // namespace llvm::cas

OnDiskActionCache::OnDiskActionCache(
    std::unique_ptr<ondisk::OnDiskKeyValueDB> DB)
    : ActionCache(builtin::BuiltinCASContext::getDefaultContext()),
      DB(std::move(DB)) {}

```
- **EN**: Introduces declarations for `llvm::cas`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `llvm::cas` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 155-168
```cpp
Expected<std::unique_ptr<OnDiskActionCache>>
OnDiskActionCache::create(StringRef AbsPath) {
  std::shared_ptr<ondisk::OnDiskCASLogger> Logger;
#ifndef _WIN32
  if (Error E =
          ondisk::OnDiskCASLogger::openIfEnabled(AbsPath).moveInto(Logger))
    return std::move(E);
#endif
  std::unique_ptr<ondisk::OnDiskKeyValueDB> DB;
  if (Error E = ondisk::OnDiskKeyValueDB::open(
                    AbsPath, getHashName(), sizeof(HashType), getHashName(),
                    sizeof(DataT), /*UnifiedCache=*/nullptr, std::move(Logger))
                    .moveInto(DB))
    return std::move(E);
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 169-182
```cpp
  return std::unique_ptr<OnDiskActionCache>(
      new OnDiskActionCache(std::move(DB)));
}

Expected<std::optional<CASID>>
OnDiskActionCache::getImpl(ArrayRef<uint8_t> Key,
                           bool /*CanBeDistributed*/) const {
  std::optional<ArrayRef<char>> Val;
  if (Error E = DB->get(Key).moveInto(Val))
    return std::move(E);
  if (!Val)
    return std::nullopt;
  return CASID::create(&getContext(), toStringRef(*Val));
}
```
- **EN**: Implements logic around `unique_ptr`, `OnDiskActionCache`, `getImpl`, `get`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `unique_ptr`, `OnDiskActionCache`, `getImpl`, `get`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 183-191
```cpp

Error OnDiskActionCache::putImpl(ArrayRef<uint8_t> Key, const CASID &Result,
                                 bool /*CanBeDistributed*/) {
  auto ResultHash = Result.getHash();
  ArrayRef Expected((const char *)ResultHash.data(), ResultHash.size());
  ArrayRef<char> Observed;
  if (Error E = DB->put(Key, Expected).moveInto(Observed))
    return E;

```
- **EN**: Implements logic around `putImpl`, `getHash`, `Expected`, `put`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `putImpl`, `getHash`, `Expected`, `put` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 192-199
```cpp
  if (Expected == Observed)
    return Error::success();

  return createResultCachePoisonedError(
      Key, getContext(), Result,
      ArrayRef((const uint8_t *)Observed.data(), Observed.size()));
}

```
- **EN**: Implements logic around `success`, `createResultCachePoisonedError`, `getContext`, `ArrayRef`; this block works with hashed storage or cache state.
- **CN**: 围绕 `success`, `createResultCachePoisonedError`, `getContext`, `ArrayRef` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 200-206
```cpp
Error OnDiskActionCache::validate() const { return DB->validate(); }

UnifiedOnDiskActionCache::UnifiedOnDiskActionCache(
    std::shared_ptr<ondisk::UnifiedOnDiskCache> UniDB)
    : ActionCache(builtin::BuiltinCASContext::getDefaultContext()),
      UniDB(std::move(UniDB)) {}

```
- **EN**: Implements logic around `validate`, `UnifiedOnDiskActionCache`, `ActionCache`, `UniDB`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `validate`, `UnifiedOnDiskActionCache`, `ActionCache`, `UniDB` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 207-219
```cpp
Expected<std::optional<CASID>>
UnifiedOnDiskActionCache::getImpl(ArrayRef<uint8_t> Key,
                                  bool /*CanBeDistributed*/) const {
  std::optional<ArrayRef<char>> Val;
  if (Error E = UniDB->getKeyValueDB().get(Key).moveInto(Val))
    return std::move(E);
  if (!Val)
    return std::nullopt;
  auto ID = ondisk::UnifiedOnDiskCache::getObjectIDFromValue(*Val);
  return CASID::create(&getContext(),
                       toStringRef(UniDB->getGraphDB().getDigest(ID)));
}

```
- **EN**: Implements logic around `getImpl`, `getKeyValueDB`, `move`, `getObjectIDFromValue`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `getImpl`, `getKeyValueDB`, `move`, `getObjectIDFromValue`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 220-226
```cpp
Error UnifiedOnDiskActionCache::putImpl(ArrayRef<uint8_t> Key,
                                        const CASID &Result,
                                        bool /*CanBeDistributed*/) {
  auto Expected = UniDB->getGraphDB().getReference(Result.getHash());
  if (LLVM_UNLIKELY(!Expected))
    return Expected.takeError();

```
- **EN**: Implements logic around `putImpl`, `getGraphDB`, `takeError`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `putImpl`, `getGraphDB`, `takeError` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 227-235
```cpp
  auto Value = ondisk::UnifiedOnDiskCache::getValueFromObjectID(*Expected);
  std::optional<ArrayRef<char>> Observed;
  if (Error E = UniDB->getKeyValueDB().put(Key, Value).moveInto(Observed))
    return E;

  auto ObservedID = ondisk::UnifiedOnDiskCache::getObjectIDFromValue(*Observed);
  if (*Expected == ObservedID)
    return Error::success();

```
- **EN**: Implements logic around `getValueFromObjectID`, `getKeyValueDB`, `getObjectIDFromValue`, `success`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `getValueFromObjectID`, `getKeyValueDB`, `getObjectIDFromValue`, `success` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 236-243
```cpp
  return createResultCachePoisonedError(
      Key, getContext(), Result, UniDB->getGraphDB().getDigest(ObservedID));
}

Error UnifiedOnDiskActionCache::validate() const {
  return UniDB->validateActionCache();
}

```
- **EN**: Implements logic around `createResultCachePoisonedError`, `getContext`, `validate`, `validateActionCache`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `createResultCachePoisonedError`, `getContext`, `validate`, `validateActionCache` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 244-252
```cpp
Expected<std::unique_ptr<ActionCache>>
cas::createOnDiskActionCache(StringRef Path) {
#if LLVM_ENABLE_ONDISK_CAS
  return OnDiskActionCache::create(Path);
#else
  return createStringError(inconvertibleErrorCode(), "OnDiskCache is disabled");
#endif
}

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 253-257
```cpp
std::unique_ptr<ActionCache>
cas::builtin::createActionCacheFromUnifiedOnDiskCache(
    std::shared_ptr<ondisk::UnifiedOnDiskCache> UniDB) {
  return std::make_unique<UnifiedOnDiskActionCache>(std::move(UniDB));
}
```
- **EN**: Implements logic around `createActionCacheFromUnifiedOnDiskCache`, `make_unique`; this block works with hashed storage or cache state.
- **CN**: 围绕 `createActionCacheFromUnifiedOnDiskCache`, `make_unique` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

## Key Concepts / 关键概念

- **Content-addressable storage / 内容寻址存储**:
  - **EN**: Represents objects by hash and manages in-memory or on-disk persistence.
  - **CN**: 通过哈希表示对象，并管理内存或磁盘持久化。
- **Structured error handling / 结构化错误处理**:
  - **EN**: Uses `Expected`, `Error`, or related helpers to make failures explicit.
  - **CN**: 使用 `Expected`、`Error` 或相关辅助工具显式表示失败。
- **Concurrency or parallel work / 并发或并行工作**:
  - **EN**: Coordinates tasks that may execute concurrently or partition work.
  - **CN**: 协调可能并发执行或分片处理的任务。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `BuiltinCAS.h`, `llvm/ADT/TrieRawHashMap.h`, `llvm/CAS/ActionCache.h`, `llvm/CAS/OnDiskCASLogger.h`, `llvm/CAS/OnDiskKeyValueDB.h`, `llvm/CAS/UnifiedOnDiskCache.h`, `llvm/Config/llvm-config.h`, `llvm/Support/BLAKE3.h`, `llvm/Support/Errc.h`
- **Subsystem categories / 子系统类别**: content-addressable storage interfaces / 内容寻址存储接口 (4), support-library helpers / Support 库辅助功能 (2), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1), LLVM build configuration details / LLVM 构建配置细节 (1)
