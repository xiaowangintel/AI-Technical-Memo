# StorageUniquer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Support/StorageUniquer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements shared support utilities used across MLIR libraries and tools.
  - **CN**: 实现供 MLIR 库与工具共享使用的支持工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===- StorageUniquer.cpp - Common Storage Class Uniquer ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Support/StorageUniquer.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Support/StorageUniquer.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Support/StorageUniquer.h`。

### Lines 11-20
```cpp
#include "mlir/Support/LLVM.h"
#include "mlir/Support/ThreadLocalCache.h"
#include "mlir/Support/TypeID.h"
#include "llvm/Support/RWMutex.h"

using namespace mlir;
using namespace mlir::detail;

namespace {
/// This class represents a uniquer for storage instances of a specific type
```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Support/LLVM.h`, `mlir/Support/ThreadLocalCache.h`, `mlir/Support/TypeID.h`, `llvm/Support/RWMutex.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Support/LLVM.h`, `mlir/Support/ThreadLocalCache.h`, `mlir/Support/TypeID.h`, `llvm/Support/RWMutex.h`。

### Lines 21-30
```cpp
/// that has parametric storage. It contains all of the necessary data to unique
/// storage instances in a thread safe way. This allows for the main uniquer to
/// bucket each of the individual sub-types removing the need to lock the main
/// uniquer itself.
class ParametricStorageUniquer {
public:
  using BaseStorage = StorageUniquer::BaseStorage;
  using StorageAllocator = StorageUniquer::StorageAllocator;

  /// A lookup key for derived instances of storage objects.
```
- **EN**: Introduces declarations for `ParametricStorageUniquer`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `ParametricStorageUniquer` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 31-40
```cpp
  struct LookupKey {
    /// The known hash value of the key.
    unsigned hashValue;

    /// An equality function for comparing with an existing storage instance.
    function_ref<bool(const BaseStorage *)> isEqual;
  };

private:
  /// A utility wrapper object representing a hashed storage object. This class
```
- **EN**: Introduces declarations for `LookupKey`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `LookupKey` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 41-50
```cpp
  /// contains a storage object and an existing computed hash value.
  struct HashedStorage {
    HashedStorage(unsigned hashValue = 0, BaseStorage *storage = nullptr)
        : hashValue(hashValue), storage(storage) {}
    unsigned hashValue;
    BaseStorage *storage;
  };

  /// Storage info for derived TypeStorage objects.
  struct StorageKeyInfo {
```
- **EN**: Introduces declarations for `HashedStorage`, `StorageKeyInfo`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `HashedStorage`、`StorageKeyInfo` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 51-64
```cpp
    static inline HashedStorage getEmptyKey() {
      return HashedStorage(0, DenseMapInfo<BaseStorage *>::getEmptyKey());
    }
    static inline HashedStorage getTombstoneKey() {
      return HashedStorage(0, DenseMapInfo<BaseStorage *>::getTombstoneKey());
    }

    static inline unsigned getHashValue(const HashedStorage &key) {
      return key.hashValue;
    }
    static inline unsigned getHashValue(const LookupKey &key) {
      return key.hashValue;
    }

```
- **EN**: Implements logic around `getEmptyKey`, `HashedStorage`, `getTombstoneKey`, `getHashValue`.
- **CN**: 围绕 `getEmptyKey`、`HashedStorage`、`getTombstoneKey`、`getHashValue` 实现具体逻辑。

### Lines 65-77
```cpp
    static inline bool isEqual(const HashedStorage &lhs,
                               const HashedStorage &rhs) {
      return lhs.storage == rhs.storage;
    }
    static inline bool isEqual(const LookupKey &lhs, const HashedStorage &rhs) {
      if (isEqual(rhs, getEmptyKey()) || isEqual(rhs, getTombstoneKey()))
        return false;
      // Invoke the equality function on the lookup key.
      return lhs.isEqual(rhs.storage);
    }
  };
  using StorageTypeSet = DenseSet<HashedStorage, StorageKeyInfo>;

```
- **EN**: Implements logic around `isEqual`.
- **CN**: 围绕 `isEqual` 实现具体逻辑。

### Lines 78-90
```cpp
  /// This class represents a single shard of the uniquer. The uniquer uses a
  /// set of shards to allow for multiple threads to create instances with less
  /// lock contention.
  struct Shard {
    /// The set containing the allocated storage instances.
    StorageTypeSet instances;

#if LLVM_ENABLE_THREADS != 0
    /// A mutex to keep uniquing thread-safe.
    llvm::sys::SmartRWMutex<true> mutex;
#endif
  };

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 91-101
```cpp
  /// Get or create an instance of a param derived type in an thread-unsafe
  /// fashion.
  BaseStorage *getOrCreateUnsafe(Shard &shard, LookupKey &key,
                                 function_ref<BaseStorage *()> ctorFn) {
    auto existing = shard.instances.insert_as({key.hashValue}, key);
    BaseStorage *&storage = existing.first->storage;
    if (existing.second)
      storage = ctorFn();
    return storage;
  }

```
- **EN**: Implements logic around `getOrCreateUnsafe`, `insert_as`, `ctorFn`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getOrCreateUnsafe`、`insert_as`、`ctorFn` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 102-112
```cpp
  /// Destroy all of the storage instances within the given shard.
  void destroyShardInstances(Shard &shard) {
    if (!destructorFn)
      return;
    for (HashedStorage &instance : shard.instances)
      destructorFn(instance.storage);
  }

public:
#if LLVM_ENABLE_THREADS != 0
  /// Initialize the storage uniquer with a given number of storage shards to
```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 113-132
```cpp
  /// use. The provided shard number is required to be a valid power of 2. The
  /// destructor function is used to destroy any allocated storage instances.
  ParametricStorageUniquer(function_ref<void(BaseStorage *)> destructorFn,
                           size_t numShards = 8)
      : shards(new std::atomic<Shard *>[numShards]), numShards(numShards),
        destructorFn(destructorFn) {
    assert(llvm::isPowerOf2_64(numShards) &&
           "the number of shards is required to be a power of 2");
    for (size_t i = 0; i < numShards; i++)
      shards[i].store(nullptr, std::memory_order_relaxed);
  }
  ~ParametricStorageUniquer() {
    // Free all of the allocated shards.
    for (size_t i = 0; i != numShards; ++i) {
      if (Shard *shard = shards[i].load()) {
        destroyShardInstances(*shard);
        delete shard;
      }
    }
  }
```
- **EN**: Implements logic around `ParametricStorageUniquer`, `shards`, `destructorFn`, `assert`, and 4 more symbols.
- **CN**: 围绕 `ParametricStorageUniquer`、`shards`、`destructorFn`、`assert` 等另外 4 个符号 实现具体逻辑。

### Lines 133-147
```cpp
  /// Get or create an instance of a parametric type.
  BaseStorage *getOrCreate(bool threadingIsEnabled, unsigned hashValue,
                           function_ref<bool(const BaseStorage *)> isEqual,
                           function_ref<BaseStorage *()> ctorFn) {
    Shard &shard = getShard(hashValue);
    ParametricStorageUniquer::LookupKey lookupKey{hashValue, isEqual};
    if (!threadingIsEnabled)
      return getOrCreateUnsafe(shard, lookupKey, ctorFn);

    // Check for a instance of this object in the local cache.
    auto localIt = localCache->insert_as({hashValue}, lookupKey);
    BaseStorage *&localInst = localIt.first->storage;
    if (localInst)
      return localInst;

```
- **EN**: Implements logic around `getOrCreate`, `function_ref`, `getShard`, `getOrCreateUnsafe`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getOrCreate`、`function_ref`、`getShard`、`getOrCreateUnsafe` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 148-161
```cpp
    // Check for an existing instance in read-only mode.
    {
      llvm::sys::SmartScopedReader<true> typeLock(shard.mutex);
      auto it = shard.instances.find_as(lookupKey);
      if (it != shard.instances.end())
        return localInst = it->storage;
    }

    // Acquire a writer-lock so that we can safely create the new storage
    // instance.
    llvm::sys::SmartScopedWriter<true> typeLock(shard.mutex);
    return localInst = getOrCreateUnsafe(shard, lookupKey, ctorFn);
  }

```
- **EN**: Implements logic around `typeLock`, `find_as`, `end`, `getOrCreateUnsafe`.
- **CN**: 围绕 `typeLock`、`find_as`、`end`、`getOrCreateUnsafe` 实现具体逻辑。

### Lines 162-176
```cpp
  /// Run a mutation function on the provided storage object in a thread-safe
  /// way.
  LogicalResult mutate(bool threadingIsEnabled, BaseStorage *storage,
                       function_ref<LogicalResult()> mutationFn) {
    if (!threadingIsEnabled)
      return mutationFn();

    // Get a shard to use for mutating this storage instance. It doesn't need to
    // be the same shard as the original allocation, but does need to be
    // deterministic.
    Shard &shard = getShard(llvm::hash_value(storage));
    llvm::sys::SmartScopedWriter<true> lock(shard.mutex);
    return mutationFn();
  }

```
- **EN**: Implements logic around `mutate`, `function_ref`, `mutationFn`, `getShard`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `mutate`、`function_ref`、`mutationFn`、`getShard` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 177-187
```cpp
private:
  /// Return the shard used for the given hash value.
  Shard &getShard(unsigned hashValue) {
    // Get a shard number from the provided hashvalue.
    unsigned shardNum = hashValue & (numShards - 1);

    // Try to acquire an already initialized shard.
    Shard *shard = shards[shardNum].load(std::memory_order_acquire);
    if (shard)
      return *shard;

```
- **EN**: Implements logic around `getShard`, `load`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getShard`、`load` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 188-197
```cpp
    // Otherwise, try to allocate a new shard.
    Shard *newShard = new Shard();
    if (shards[shardNum].compare_exchange_strong(shard, newShard))
      return *newShard;

    // If one was allocated before we can initialize ours, delete ours.
    delete newShard;
    return *shard;
  }

```
- **EN**: Implements logic around `Shard`, `compare_exchange_strong`.
- **CN**: 围绕 `Shard`、`compare_exchange_strong` 实现具体逻辑。

### Lines 198-207
```cpp
  /// A thread local cache for storage objects. This helps to reduce the lock
  /// contention when an object already existing in the cache.
  ThreadLocalCache<StorageTypeSet> localCache;

  /// A set of uniquer shards to allow for further bucketing accesses for
  /// instances of this storage type. Each shard is lazily initialized to reduce
  /// the overhead when only a small amount of shards are in use.
  std::unique_ptr<std::atomic<Shard *>[]> shards;

  /// The number of available shards.
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 208-221
```cpp
  size_t numShards;

  /// Function to used to destruct any allocated storage instances.
  function_ref<void(BaseStorage *)> destructorFn;

#else
  /// If multi-threading is disabled, ignore the shard parameter as we will
  /// always use one shard. The destructor function is used to destroy any
  /// allocated storage instances.
  ParametricStorageUniquer(function_ref<void(BaseStorage *)> destructorFn,
                           size_t numShards = 0)
      : destructorFn(destructorFn) {}
  ~ParametricStorageUniquer() { destroyShardInstances(shard); }

```
- **EN**: Implements logic around `function_ref`, `ParametricStorageUniquer`, `destructorFn`, `~ParametricStorageUniquer`.
- **CN**: 围绕 `function_ref`、`ParametricStorageUniquer`、`destructorFn`、`~ParametricStorageUniquer` 实现具体逻辑。

### Lines 222-231
```cpp
  /// Get or create an instance of a parametric type.
  BaseStorage *
  getOrCreate(bool threadingIsEnabled, unsigned hashValue,
              function_ref<bool(const BaseStorage *)> isEqual,
              function_ref<BaseStorage *()> ctorFn) {
    ParametricStorageUniquer::LookupKey lookupKey{hashValue, isEqual};
    return getOrCreateUnsafe(shard, lookupKey, ctorFn);
  }
  /// Run a mutation function on the provided storage object in a thread-safe
  /// way.
```
- **EN**: Implements logic around `getOrCreate`, `function_ref`, `getOrCreateUnsafe`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getOrCreate`、`function_ref`、`getOrCreateUnsafe` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 232-241
```cpp
  LogicalResult
  mutate(bool threadingIsEnabled, BaseStorage *storage,
         function_ref<LogicalResult()> mutationFn) {
    return mutationFn();
  }

private:
  /// The main uniquer shard that is used for allocating storage instances.
  Shard shard;

```
- **EN**: Implements logic around `mutate`, `function_ref`, `mutationFn`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `mutate`、`function_ref`、`mutationFn` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 242-251
```cpp
  /// Function to used to destruct any allocated storage instances.
  function_ref<void(BaseStorage *)> destructorFn;
#endif
};
} // namespace

namespace mlir {
namespace detail {
/// This is the implementation of the StorageUniquer class.
struct StorageUniquerImpl {
```
- **EN**: Introduces declarations for `mlir`, `detail`, `StorageUniquerImpl`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `mlir`、`detail`、`StorageUniquerImpl` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 252-261
```cpp
  using BaseStorage = StorageUniquer::BaseStorage;
  using StorageAllocator = StorageUniquer::StorageAllocator;

  //===--------------------------------------------------------------------===//
  // Parametric Storage
  //===--------------------------------------------------------------------===//

  /// Check if an instance of a parametric storage class exists.
  bool hasParametricStorage(TypeID id) { return parametricUniquers.count(id); }

```
- **EN**: Implements logic around `hasParametricStorage`.
- **CN**: 围绕 `hasParametricStorage` 实现具体逻辑。

### Lines 262-274
```cpp
  /// Get or create an instance of a parametric type.
  BaseStorage *
  getOrCreate(TypeID id, unsigned hashValue,
              function_ref<bool(const BaseStorage *)> isEqual,
              function_ref<BaseStorage *(StorageAllocator &)> ctorFn) {
    assert(parametricUniquers.count(id) &&
           "creating unregistered storage instance");
    ParametricStorageUniquer &storageUniquer = *parametricUniquers[id];
    return storageUniquer.getOrCreate(
        threadingIsEnabled, hashValue, isEqual,
        [&] { return ctorFn(getThreadSafeAllocator()); });
  }

```
- **EN**: Implements logic around `getOrCreate`, `function_ref`, `assert`, `ctorFn`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getOrCreate`、`function_ref`、`assert`、`ctorFn` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 275-287
```cpp
  /// Run a mutation function on the provided storage object in a thread-safe
  /// way.
  LogicalResult
  mutate(TypeID id, BaseStorage *storage,
         function_ref<LogicalResult(StorageAllocator &)> mutationFn) {
    assert(parametricUniquers.count(id) &&
           "mutating unregistered storage instance");
    ParametricStorageUniquer &storageUniquer = *parametricUniquers[id];
    return storageUniquer.mutate(threadingIsEnabled, storage, [&] {
      return mutationFn(getThreadSafeAllocator());
    });
  }

```
- **EN**: Implements logic around `mutate`, `function_ref`, `assert`, `mutationFn`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `mutate`、`function_ref`、`assert`、`mutationFn` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 288-299
```cpp
  /// Return an allocator that can be used to safely allocate instances on the
  /// current thread.
  StorageAllocator &getThreadSafeAllocator() {
#if LLVM_ENABLE_THREADS != 0
    if (!threadingIsEnabled)
      return allocator;

    // If the allocator has not been initialized, create a new one.
    StorageAllocator *&threadAllocator = threadSafeAllocator.get();
    if (!threadAllocator) {
      threadAllocator = new StorageAllocator();

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 300-312
```cpp
      // Record this allocator, given that we don't want it to be destroyed when
      // the thread dies.
      llvm::sys::SmartScopedLock<true> lock(threadAllocatorMutex);
      threadAllocators.push_back(
          std::unique_ptr<StorageAllocator>(threadAllocator));
    }

    return *threadAllocator;
#else
    return allocator;
#endif
  }

```
- **EN**: Implements logic around `lock`, `push_back`, `unique_ptr`.
- **CN**: 围绕 `lock`、`push_back`、`unique_ptr` 实现具体逻辑。

### Lines 313-323
```cpp
  //===--------------------------------------------------------------------===//
  // Singleton Storage
  //===--------------------------------------------------------------------===//

  /// Get or create an instance of a singleton storage class.
  BaseStorage *getSingleton(TypeID id) {
    BaseStorage *singletonInstance = singletonInstances[id];
    assert(singletonInstance && "expected singleton instance to exist");
    return singletonInstance;
  }

```
- **EN**: Implements logic around `getSingleton`, `assert`.
- **CN**: 围绕 `getSingleton`、`assert` 实现具体逻辑。

### Lines 324-333
```cpp
  /// Check if an instance of a singleton storage class exists.
  bool hasSingleton(TypeID id) const { return singletonInstances.count(id); }

  //===--------------------------------------------------------------------===//
  // Instance Storage
  //===--------------------------------------------------------------------===//

#if LLVM_ENABLE_THREADS != 0
  /// A thread local set of allocators used for uniquing parametric instances,
  /// or other data allocated in thread volatile situations.
```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 334-343
```cpp
  ThreadLocalCache<StorageAllocator *> threadSafeAllocator;

  /// All of the allocators that have been created for thread based allocation.
  std::vector<std::unique_ptr<StorageAllocator>> threadAllocators;

  /// A mutex used for safely adding a new thread allocator.
  llvm::sys::SmartMutex<true> threadAllocatorMutex;
#endif

  /// Main allocator used for uniquing singleton instances, and other state when
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 344-354
```cpp
  /// thread safety is guaranteed.
  StorageAllocator allocator;

  /// Map of type ids to the storage uniquer to use for registered objects.
  DenseMap<TypeID, std::unique_ptr<ParametricStorageUniquer>>
      parametricUniquers;

  /// Map of type ids to a singleton instance when the storage class is a
  /// singleton.
  DenseMap<TypeID, BaseStorage *> singletonInstances;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 355-364
```cpp
  /// Flag specifying if multi-threading is enabled within the uniquer.
  bool threadingIsEnabled = true;
};
} // namespace detail
} // namespace mlir

StorageUniquer::StorageUniquer() : impl(new StorageUniquerImpl()) {}
StorageUniquer::~StorageUniquer() = default;

/// Set the flag specifying if multi-threading is disabled within the uniquer.
```
- **EN**: Implements logic around `StorageUniquer`, `~StorageUniquer`.
- **CN**: 围绕 `StorageUniquer`、`~StorageUniquer` 实现具体逻辑。

### Lines 365-377
```cpp
void StorageUniquer::disableMultithreading(bool disable) {
  impl->threadingIsEnabled = !disable;
}

/// Implementation for getting/creating an instance of a derived type with
/// parametric storage.
auto StorageUniquer::getParametricStorageTypeImpl(
    TypeID id, unsigned hashValue,
    function_ref<bool(const BaseStorage *)> isEqual,
    function_ref<BaseStorage *(StorageAllocator &)> ctorFn) -> BaseStorage * {
  return impl->getOrCreate(id, hashValue, isEqual, ctorFn);
}

```
- **EN**: Implements logic around `disableMultithreading`, `getParametricStorageTypeImpl`, `function_ref`, `getOrCreate`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `disableMultithreading`、`getParametricStorageTypeImpl`、`function_ref`、`getOrCreate` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 378-387
```cpp
/// Implementation for registering an instance of a derived type with
/// parametric storage.
void StorageUniquer::registerParametricStorageTypeImpl(
    TypeID id, function_ref<void(BaseStorage *)> destructorFn) {
  impl->parametricUniquers.try_emplace(
      id, std::make_unique<ParametricStorageUniquer>(destructorFn));
}

/// Implementation for getting an instance of a derived type with default
/// storage.
```
- **EN**: Implements logic around `registerParametricStorageTypeImpl`, `function_ref`, `try_emplace`, `make_unique`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `registerParametricStorageTypeImpl`、`function_ref`、`try_emplace`、`make_unique` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 388-397
```cpp
auto StorageUniquer::getSingletonImpl(TypeID id) -> BaseStorage * {
  return impl->getSingleton(id);
}

/// Test is the storage singleton is initialized.
bool StorageUniquer::isSingletonStorageInitialized(TypeID id) {
  return impl->hasSingleton(id);
}

/// Test is the parametric storage is initialized.
```
- **EN**: Implements logic around `getSingletonImpl`, `getSingleton`, `isSingletonStorageInitialized`, `hasSingleton`.
- **CN**: 围绕 `getSingletonImpl`、`getSingleton`、`isSingletonStorageInitialized`、`hasSingleton` 实现具体逻辑。

### Lines 398-410
```cpp
bool StorageUniquer::isParametricStorageInitialized(TypeID id) {
  return impl->hasParametricStorage(id);
}

/// Implementation for registering an instance of a derived type with default
/// storage.
void StorageUniquer::registerSingletonImpl(
    TypeID id, function_ref<BaseStorage *(StorageAllocator &)> ctorFn) {
  assert(!impl->singletonInstances.count(id) &&
         "storage class already registered");
  impl->singletonInstances.try_emplace(id, ctorFn(impl->allocator));
}

```
- **EN**: Implements logic around `isParametricStorageInitialized`, `hasParametricStorage`, `registerSingletonImpl`, `assert`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `isParametricStorageInitialized`、`hasParametricStorage`、`registerSingletonImpl`、`assert` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 411-416
```cpp
/// Implementation for mutating an instance of a derived storage.
LogicalResult StorageUniquer::mutateImpl(
    TypeID id, BaseStorage *storage,
    function_ref<LogicalResult(StorageAllocator &)> mutationFn) {
  return impl->mutate(id, storage, mutationFn);
}
```
- **EN**: Implements logic around `mutateImpl`, `function_ref`, `mutate`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `mutateImpl`、`function_ref`、`mutate` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

## Key Concepts / 关键概念

- **Shared support utilities / 共享支持工具**:
  - **EN**: Provides reusable helpers that are intentionally lower level than dialect-specific logic.
  - **CN**: 提供刻意保持在方言逻辑之下层级的可复用辅助能力。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Runtime interop / 运行时互操作**:
  - **EN**: The code exposes or consumes ABI-level hooks used by generated code at runtime.
  - **CN**: 代码暴露或消费运行时由生成代码使用的 ABI 级钩子。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Support/StorageUniquer.h`, `mlir/Support/LLVM.h`, `mlir/Support/ThreadLocalCache.h`, `mlir/Support/TypeID.h`, `llvm/Support/RWMutex.h`
- **Subsystem categories / 子系统类别**: shared MLIR support helpers / 共享的 MLIR 支持工具 (4), LLVM support-library helpers / LLVM Support 库辅助工具 (1)
