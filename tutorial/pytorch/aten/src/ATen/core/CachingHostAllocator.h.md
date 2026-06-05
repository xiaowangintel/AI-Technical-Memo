# CachingHostAllocator.h — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/core/CachingHostAllocator.h`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file declares core ATen/C10 abstractions, with primary focus on `or`, `HostBlock`, `alignas`.
- 用途（中文）: 该文件声明ATen/C10 核心抽象，核心关注对象是 `or`, `HostBlock`, `alignas`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
#pragma once

#include <c10/core/Allocator.h>
#include <c10/core/AllocatorConfig.h>
#include <c10/core/Stream.h>
#include <c10/core/thread_pool.h>
#include <c10/util/flat_hash_map.h>
#include <c10/util/llvmMathExtras.h>
#include <iostream>
#include <optional>

#include <deque>
#include <vector>
#include <mutex>
#include <shared_mutex>

C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wunused-parameter")
namespace at {

using c10::CachingAllocator::Stat;
using c10::CachingAllocator::DurationStat;

/**
 * HostBlock is typically a fundamental memory block used in pinned memory. It
```
- EN: Focus symbols: `at`, `C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED`. This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 关注符号：`at`, `C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED`。该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 25-50
```cpp
 * is likely related to Event and Stream of device runtime. It is probably a
 * base struct or interface that can be inherited and extended by each backend.
 */
template <typename S>
struct HostBlock {
  // constructor for search key
  HostBlock(size_t size) : size_(size) {}

  HostBlock(size_t size, void* ptr) : size_(size), ptr_(ptr) {}

  std::mutex mutex_;
  size_t size_{0}; // block size in bytes
  void* ptr_{nullptr}; // memory address
  bool allocated_{false}; // in-use flag
  size_t event_count_{0}; // number of related events
  ska::flat_hash_set<S> streams_; // streams on which the block was used
  c10::MempoolId_t owning_pool_{0,0}; // never changes after construction, so we don't need a mutex to guard this
  bool was_allocated_during_stream_capture_;
};

template <typename B>
struct alignas(hardware_destructive_interference_size) FreeBlockList {
  std::mutex mutex_;
  std::deque<B*> list_;
};

```
- EN: Focus symbols: `or`, `HostBlock`, `alignas`, `size_`, `ptr_`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`or`, `HostBlock`, `alignas`, `size_`, `ptr_`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 51-76
```cpp
namespace {
  // Max cached block sizes: (1 << MAX_SIZE_INDEX) bytes
  // NOLINTNEXTLINE(misc-definitions-in-headers)
  constexpr size_t MAX_SIZE_INDEX = 64;
}

// A large reserved pinned memory segment that is created in advance which is used
// to allocate small pinned memory requests to avoid calling into expensive APIs.
// We never free this memory and move up the pointer as we allocate new blocks
// and when blocks are freed, they are cached in the free lists.
struct PinnedReserveSegment {
  PinnedReserveSegment(void *start, size_t size) : start_(start), size_(size),
    current_ptr_(start_), initialized_(true) {}

  PinnedReserveSegment() : start_(nullptr), size_(0), current_ptr_(nullptr), initialized_(false) {}

  bool initialized() {
    return initialized_;
  }

  void* allocate(size_t bytes) {
    std::lock_guard<std::mutex> guard(mutex_);

    // Round up the requested size to 4KB boundary for all including the small ones.
    size_t rounded_bytes = (bytes + 4096 - 1) & ~(4096 - 1);

```
- EN: Focus symbols: `PinnedReserveSegment`, `start_`, `size_`, `current_ptr_`, `initialized_`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`PinnedReserveSegment`, `start_`, `size_`, `current_ptr_`, `initialized_`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 77-100
```cpp
    if (((uint8_t*)current_ptr_ + rounded_bytes) > ((uint8_t*)start_ + size_)) {
      return nullptr;
    }

    void* ptr = current_ptr_;
    current_ptr_ = (uint8_t*)current_ptr_ + rounded_bytes;
    return ptr;
  }

  bool owns(void* ptr) {
    return ptr >= start_ && ptr < (uint8_t*)start_ + size_;
  }

  std::mutex mutex_;
  void* start_;
  size_t size_;
  void* current_ptr_;
  bool initialized_;
};

// Struct containing memory allocator summary statistics for host.
struct TORCH_API HostStats {
  // COUNT: total allocations (active)
  Stat active_requests;
```
- EN: Focus symbols: `HostStats`, `owns`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`HostStats`, `owns`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 101-125
```cpp
  // SUM: bytes allocated/reserved by this memory allocator. (active)
  Stat active_bytes;
  // COUNT: total allocations (active + free)
  Stat allocations;
  // SUM: bytes allocated/reserved by this memory allocator. This accounts
  // for both free and in-use blocks.
  Stat allocated_bytes;

  // SUM: time spent in cudaHostAlloc/cudaHostRegister in microseconds
  DurationStat host_alloc_time;

  // SUM: time spent in cudaHostFree/cudaHostUnregister in microseconds
  DurationStat host_free_time;

  // COUNT: number of times cudaHostAlloc/cudaHostRegister was called because
  // the request could not be satisfied from existing free blocks.
  int64_t num_host_alloc = 0; // This is derived from segment or timing

  // COUNT: number of times cudaHostFree/cudaHostUnregister was called.
  int64_t num_host_free = 0; // This is derived from segment or timing

  // Count of cudaHostAlloc/cudaHostRegister per bucket
  std::vector<int64_t> bucket_allocation = std::vector<int64_t>(MAX_SIZE_INDEX);
};

```
- EN: This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 126-149
```cpp
// Struct containing memory allocator summary statistics for host, as they
// are staged for reporting. This is a temporary struct that is used to
// avoid locking the allocator while collecting stats.
struct alignas(hardware_destructive_interference_size) HostStatsStaged {
  std::mutex timing_mutex_;
  // COUNT: total allocations (active + free)
  // LOCK: access to this stat is protected by the allocator's blocks_mutex_
  Stat allocations;
  // SUM: bytes allocated/reserved by this memory allocator. This accounts
  // for both free and in-use blocks.
  Stat allocated_bytes;
  // COUNT: number of allocations per bucket (active)
  // LOCK: access to this stat is protected by the per bucket free_list_[index].mutex_
  std::vector<Stat> active_bucket_stats = std::vector<Stat>(MAX_SIZE_INDEX);
  // SUM: bytes of allocation per bucket (active)
  // LOCK: access to this stat is protected by the per bucket free_list_[index].mutex_
  std::vector<Stat> active_bytes_bucket_stats = std::vector<Stat>(MAX_SIZE_INDEX);
  // COUNT: number of allocations per bucket (active + free)
  // LOCK: access to this stat is protected by the per bucket free_list_[index].mutex_
  std::vector<Stat> allocation_bucket_stats = std::vector<Stat>(MAX_SIZE_INDEX);
  // SUM: bytes of allocation per bucket (active + free)
  // LOCK: access to this stat is protected by the per bucket free_list_[index].mutex_
  std::vector<Stat> allocated_bytes_bucket_stats = std::vector<Stat>(MAX_SIZE_INDEX);
  // SUM: time spent in cudaHostAlloc/cudaHostRegister
```
- EN: Focus symbols: `that`, `alignas`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`that`, `alignas`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 150-173
```cpp
  // LOCK: access to this stat is protected by the timing_mutex_
  DurationStat host_alloc_time;
  // SUM: time spent in cudaHostFree/cudaHostUnregister
  // LOCK: access to this stat is protected by the timing_mutex_
  DurationStat host_free_time;
};

/**
 * Note [HostAllocator design]
 * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 * We have three key data structures - the free list which stores blocks that
 * are not currently used, the block list which stores all blocks that have been
 * allocated, and the event queue which stores runtime events and their
 * corresponding blocks.
 *
 * Each of these are protected by a separate mutex. The key design principles
 * are to 1) only hold each mutex for the minimal amount of time possible, 2)
 * never do any possible expensive operations (such as CUDA runtime API calls)
 * while holding the lock.
 *
 * There are four public methods: allocate, free, record_event and empty_cache.
 *   1) In the allocate path, we first check to see if we can service our
 * request from this free list, and otherwise we create a new block with
 * allocate_host_memory.
```
- EN: Focus symbols: `operations`. This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 关注符号：`operations`。该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

### Lines 174-197
```cpp
 *   2) In the free path, we insert events (if required) into the event queue,
 * and if possible insert our block back into the free list. In allocate, we
 * first eagerly query events until we find one that is not ready, and insert
 * the corresponding block onto the free list if all the events recorded for a
 * block are ready.
 *   3) In the record_event path, we simply insert the given stream into the set
 * of streams tracked by the specified block. This set of streams is then
 * consumed in the free path.
 *   4) In the empty_cache path, we flush any available blocks into the free
 * list. Remove all element of free list, then remove them from block list and
 * release the associated pinned memory allocation via free_block.
 *
 * We generalize the caching host allocator into two parts: interface and
 * implementation. For any new backend looking to integrate with host allocator
 * and reuse caching mechanism, these two parts are necessary to be specialized.
 *
 * For the implementation, we provide a CachingHostAllocatorImpl struct
 * to abstract the caching mechanism. Any backend needs to provide a customized
 * implementation by specializing its own public functions and the related
 * runtime functions. Its template parameter S represents runtime Stream, E
 * denotes runtime Event, B indicates the fundamental memory block.
 *
 * For the interface, we provide a CachingHostAllocatorInterface struct as an
 * interface. Any backend needs to derive its own host allocator from this
```
- EN: Focus symbols: `as`, `events`. This block documents the design intent, assumptions, or usage notes. The comments give maintainers the context needed to interpret the following code correctly.
- CN: 关注符号：`as`, `events`。该代码块说明设计意图、前提假设或使用说明。这些注释为维护者提供理解后续代码所需的上下文。

### Lines 198-221
```cpp
 * interface. Its template parameter T refers to an implementation that
 * inherited from CachingHostAllocatorImpl.
 *
 * So this design can share the caching mechanism across each backend, and
 * provide flexibility to each backend. A backend can choose to follow this
 * implementation or reuse them by extending and overriding them as necessary.
 * Taking CUDA as an example, it specializes runtime related functions to reuse
 * the caching mechanism. Additionally, it extends the allocator's functionality
 * by adding the allocWithCudaHostRegister function to support page-locking the
 * memory range used by CUDA. Of course, you can also refer to
 * XPUCachingHostAllocator, which is a host caching allocator supported on XPU
 * backend, to implement a basic host caching allocator.
 *
 * Some of the invariants here are less strict than they could be - for example,
 * we do not enforce that free(Block* block) => block->event_count == 0. This is
 * for compatibility reasons, and we can explore enforcing these in subsequent
 * versions.
 *
 * Note that this caching host allocator does not split larger allocations into
 * smaller blocks, unlike the caching device allocator.
 *
 * In order to gather statistics about caching host allocator while minimally
 * impacting performance, we use a HostStatsStaged struct to stage the stats
 * before reporting them. This is done to avoid adding new locks to the allocator.
```
- EN: Focus symbols: `to`, `free`. This block documents the design intent, assumptions, or usage notes. The comments give maintainers the context needed to interpret the following code correctly.
- CN: 关注符号：`to`, `free`。该代码块说明设计意图、前提假设或使用说明。这些注释为维护者提供理解后续代码所需的上下文。

### Lines 222-247
```cpp
 * Collecting stats is carefully done under existing locks, and then the staged
 * stats are converted to the final stats when getStats is called. At that time
 * we hold the same locks as empty_cache, to ensure the fidelity of the stats.
 */

// Generic per-pool structures for the host caching allocator. These are
// templated so they can reference the allocator's template parameters B and E.
template <typename S_, typename E_, typename B_>
struct HostBlockPool {
  HostBlockPool() = default;
  HostBlockPool(const HostBlockPool&) = delete;
  HostBlockPool& operator=(const HostBlockPool&) = delete;

  alignas(hardware_destructive_interference_size) std::mutex blocks_mutex_;
  ska::flat_hash_set<B_*> blocks_; // all blocks in this pool
  ska::flat_hash_map<void*, B_*> ptr_to_block_;

  // Per-size free lists guarded by their own mutexes.
  alignas(hardware_destructive_interference_size) std::vector<FreeBlockList<B_>> free_list_ =
      std::vector<FreeBlockList<B_>>(MAX_SIZE_INDEX);

  // Events pending for blocks in this pool.
  alignas(hardware_destructive_interference_size) std::mutex events_mutex_;
  std::deque<std::pair<E_, B_*>> events_;
};

```
- EN: Focus symbols: `HostBlockPool`, `alignas`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`HostBlockPool`, `alignas`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 248-272
```cpp
// The HostBlockPool owned by one HostPrivatePool cannot share blocks
// with any other HostPrivatePool. This invariant is necessary to
// implement cuda graph capture correctly, which depends upon virtual
// addresses staying alive for the duration of a graph's existence.
template <typename S_, typename E_, typename B_>
struct HostPrivatePool {
  explicit HostPrivatePool(c10::MempoolId_t id_) : id(id_) {}
  HostPrivatePool(const HostPrivatePool&) = delete;
  HostPrivatePool& operator=(const HostPrivatePool&) = delete;

  c10::MempoolId_t id;
  int use_count{1};
  HostBlockPool<S_, E_, B_> blocks;
};


template <
    typename S,
    typename E,
    typename B = HostBlock<S>>
struct CachingHostAllocatorImpl {

  using BlockPool = HostBlockPool<S, E, B>;
  using PrivatePool = HostPrivatePool<S, E, B>;

```
- EN: Focus symbols: `HostPrivatePool`, `CachingHostAllocatorImpl`, `BlockPool`, `PrivatePool`, `id`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`HostPrivatePool`, `CachingHostAllocatorImpl`, `BlockPool`, `PrivatePool`, `id`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 273-298
```cpp
  virtual ~CachingHostAllocatorImpl() {
    if (active_) {
      active_ = false;
      getBackgroundThreadPool()->waitWorkComplete();
    }
  }

  // return data_ptr and block pair.
  virtual std::pair<void*, void*> allocate(size_t size) {
    if (size == 0) {
      return {nullptr, nullptr};
    }

    auto&& [mempool_id, pool] = get_allocation_pool_for_current_stream();

    // If we are using background threads, we can process events in the
    // background.
    // In the case of a non-default pool, we never use the background
    // thread. Since allocations happen only during stream capture
    // time and there are no events to process anyway, speeding up
    // event processing with a helper thread is not helpful.
    if (!(pinned_use_background_threads() &&
          mempool_id.first == 0 && mempool_id.second == 0)) {
      process_events(pool);
    }

```
- EN: Focus symbols: `~CachingHostAllocatorImpl`, `getBackgroundThreadPool`, `waitWorkComplete`, `allocate`, `get_allocation_pool_for_current_stream`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`~CachingHostAllocatorImpl`, `getBackgroundThreadPool`, `waitWorkComplete`, `allocate`, `get_allocation_pool_for_current_stream`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 299-322
```cpp
    // Round up the allocation to the nearest power of two to improve reuse.
    // These power of two sizes are also used to index into the free list.
    size_t roundSize = c10::llvm::PowerOf2Ceil(size);

    // First, try to allocate from the free list of the chosen pool
    auto* block = get_free_block(roundSize, pool);
    if (block) {
      block->was_allocated_during_stream_capture_ = current_stream_is_capturing_fast_path();
      return {block->ptr_, reinterpret_cast<void*>(block)};
    }

    // Check in the recently freed blocks with pending events to see if we
    // can reuse them. Call get_free_block again after processing events
    if (pinned_use_background_threads()) {
      // Launch the background thread and process events in a loop.
      static bool background_thread_flag [[maybe_unused]] = [this] {
        active_ = true;
        getBackgroundThreadPool()->run([&]() {
          while (active_) {
            // Background thread conservatively processes default pool
            // events only. Private pools never use a background
            // thread because cuda stream capture does not benefit
            // from asynchronous event processing.
            process_events(default_pool_);
```
- EN: Focus symbols: `PowerOf2Ceil`, `get_free_block`, `current_stream_is_capturing_fast_path`, `pinned_use_background_threads`, `getBackgroundThreadPool`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`PowerOf2Ceil`, `get_free_block`, `current_stream_is_capturing_fast_path`, `pinned_use_background_threads`, `getBackgroundThreadPool`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 323-348
```cpp
            std::this_thread::sleep_for(std::chrono::microseconds(100));
          }
        });
        return true;
      }();
    }

    // Slow path: if we can't allocate from the cached free list, we need
    // to create a new block.
    void* ptr = nullptr;
    allocate_host_memory(roundSize, &ptr);

    // Then, create a new block.
    block = new B(roundSize, ptr);
    block->allocated_ = true;
    block->owning_pool_ = mempool_id;
    block->was_allocated_during_stream_capture_ = current_stream_is_capturing_fast_path();
    add_allocated_block(block, pool);
    return {block->ptr_, reinterpret_cast<void*>(block)};
  }

  virtual void free(void* ctx) {
    if (!ctx) {
      return;
    }

```
- EN: Focus symbols: `sleep_for`, `microseconds`, `allocate_host_memory`, `B`, `current_stream_is_capturing_fast_path`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`sleep_for`, `microseconds`, `allocate_host_memory`, `B`, `current_stream_is_capturing_fast_path`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 349-372
```cpp
    // Note: we can assume that free is correctly paired with alloc, and thus we
    // do not need to look up the ctx in blocks_.
    auto* block = reinterpret_cast<B*>(ctx);

    std::optional<std::vector<E>> events;
    ska::flat_hash_set<S> streams;
    bool allocated_during_capture = false;
    {
      std::lock_guard<std::mutex> g(block->mutex_);
      block->allocated_ = false;
      allocated_during_capture = block->was_allocated_during_stream_capture_;
      if (block->streams_.empty()) {
        TORCH_INTERNAL_ASSERT(block->event_count_ == 0);
      } else {
        events = std::vector<E>();
        events->reserve(block->streams_.size());
        block->event_count_ += block->streams_.size();
        // Move out streams to avoid holding the mutex during event recording
        streams = std::move(block->streams_);
        block->streams_.clear();
      }
    }

    // cudaEventRecord() does not work for indicating when a block can
```
- EN: Focus symbols: `g`, `empty`, `TORCH_INTERNAL_ASSERT`, `reserve`, `size`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`g`, `empty`, `TORCH_INTERNAL_ASSERT`, `reserve`, `size`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 373-397
```cpp
    // be reused while stream capture is happening, since no actual
    // GPU work happens during stream capture.
    if (!allocated_during_capture) {
      // Event recording must be done outside the mutex to avoid potential
      // deadlocks (e.g., when Python GIL is involved)
      for (auto stream : streams) {
        record_stream(events, stream);
      }
    }

    if (!events.has_value()) {
      auto& pool = pool_from_block(block);
      auto index = size_index(block->size_);
      std::lock_guard<std::mutex> g(pool.free_list_[index].mutex_);
      pool.free_list_[index].list_.push_back(block);
    } else if (allocated_during_capture) {
      // pass: No events are ever recorded during stream capture.

      // If the block was ever used, block->event_count_ will be above
      // 0 and thus can never be recycled by
      // process_events_for_specific_size. Thus, this block will never
      // be returned again. "Leaking" memory like this is intentional
      // to avoid subtle cuda graph problems described here:
      // https://github.com/pytorch/pytorch/pull/161583#issuecomment-3229885771

```
- EN: Focus symbols: `record_stream`, `has_value`, `pool_from_block`, `size_index`, `g`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`record_stream`, `has_value`, `pool_from_block`, `size_index`, `g`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 398-421
```cpp
      // Otherwise, if the block was never used, block->event_count_
      // will be 0 and thus process_events_for_specific_size can
      // return this block.
    } else {
      // restore these events that record by used streams.
      auto& pool = pool_from_block(block);
      std::lock_guard<std::mutex> g(pool.events_mutex_);
      for (auto&& event : *events) {
        pool.events_.emplace_front(std::move(event), block);
      }
    }
  }

  virtual bool record_event(void* ptr, void* ctx, c10::Stream s) {
    B *block = nullptr;
    if (block_exists(ctx)) {
      block = reinterpret_cast<B*>(ctx);
    } else {
      block = get_block_from_ptr(ptr);
    }
    if (block == nullptr) {
      return false;
    }
    S stream = S(s);
```
- EN: Focus symbols: `pool_from_block`, `g`, `emplace_front`, `move`, `record_event`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`pool_from_block`, `g`, `emplace_front`, `move`, `record_event`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 422-445
```cpp
    std::lock_guard<std::mutex> gb(block->mutex_);
    TORCH_INTERNAL_ASSERT(block->allocated_);
    block->streams_.insert(stream);
    return true;
  }

  void free_from_pool(BlockPool &pool) {
    for (size_t i = 0; i < pool.free_list_.size(); ++i) {
      std::lock(pool.free_list_[i].mutex_, pool.blocks_mutex_);
      std::lock_guard<std::mutex> gf(pool.free_list_[i].mutex_, std::adopt_lock);
      std::lock_guard<std::mutex> gb(pool.blocks_mutex_, std::adopt_lock);

      std::vector<B*> blocks_to_remove(pool.free_list_[i].list_.begin(),
                                       pool.free_list_[i].list_.end());
      pool.free_list_[i].list_.clear();

      for (auto* block : blocks_to_remove) {
        pool.blocks_.erase(block);
        pool.ptr_to_block_.erase(block->ptr_);
        auto index = size_index(block->size_);
        free_block(block);
        stats_.allocations.decrease(1);
        stats_.allocated_bytes.decrease(block->size_);
        stats_.allocation_bucket_stats[index].decrease(1);
```
- EN: Focus symbols: `gb`, `TORCH_INTERNAL_ASSERT`, `insert`, `free_from_pool`, `size`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`gb`, `TORCH_INTERNAL_ASSERT`, `insert`, `free_from_pool`, `size`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 446-472
```cpp
        stats_.allocated_bytes_bucket_stats[index].decrease(block->size_);
        delete block;
      }
    }
  }

  // TODO: Make this take a pool id like in CUDACachingAllocator
  virtual void empty_cache() {
    process_events(default_pool_);
    free_from_pool(default_pool_);

    {
      std::unique_lock<std::shared_mutex> lg(instance_mutex_);
      for (auto it = graph_pools_freeable_.begin(); it != graph_pools_freeable_.end();) {
        process_events(it->second->blocks);
        free_from_pool(it->second->blocks);
        if (it->second->blocks.blocks_.empty()) {
          auto erase_count = graph_pools_.erase(it->first);
          TORCH_INTERNAL_ASSERT(erase_count == 1);
          it = graph_pools_freeable_.erase(it);
        } else {
          ++it;
        }
      }
    }
  }

```
- EN: Focus symbols: `decrease`, `empty_cache`, `process_events`, `free_from_pool`, `lg`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`decrease`, `empty_cache`, `process_events`, `free_from_pool`, `lg`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 473-496
```cpp
  inline size_t size_index(size_t size) {
    return c10::llvm::Log2_64_Ceil(size);
  }

  virtual bool pinned_use_background_threads() {
    return c10::CachingAllocator::AcceleratorAllocatorConfig::
        pinned_use_background_threads();
  }

  virtual void copy_data(void* dest [[maybe_unused]], const void* src [[maybe_unused]], std::size_t count [[maybe_unused]]) const {
    TORCH_CHECK_NOT_IMPLEMENTED(false, "Not implemented for copy_data");
  }

  HostStats getStats() {
    HostStats stats;

    // To keep getStats lightweight we do *not* flush any available blocks
    // into the free_list. This may skew the stats a bit.

    auto add_bucket_stats = [](Stat& accumulator, const Stat& other) {
      accumulator.allocated += other.allocated;
      accumulator.current += other.current;
      accumulator.freed += other.freed;
      // Since peaks are measured per bucket independently, we add them up
```
- EN: Focus symbols: `size_index`, `Log2_64_Ceil`, `pinned_use_background_threads`, `copy_data`, `TORCH_CHECK_NOT_IMPLEMENTED`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`size_index`, `Log2_64_Ceil`, `pinned_use_background_threads`, `copy_data`, `TORCH_CHECK_NOT_IMPLEMENTED`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 497-522
```cpp
      // to estimate the total peak. This is not strictly correct, but it is
      // the best approximation we can get after the fact.
      accumulator.peak += other.peak;
    };

    // Accurate reading of memory stats requires concurrently holding both the
    // free list mutexes and the blocks mutex. Previously, this was only done in
    // empty_cache function.
    for (size_t i = 0; i < default_pool_.free_list_.size(); ++i) {
      std::lock(
          default_pool_.free_list_[i].mutex_, default_pool_.blocks_mutex_);
      std::lock_guard<std::mutex> gf(
          default_pool_.free_list_[i].mutex_, std::adopt_lock);
      std::lock_guard<std::mutex> gb(
          default_pool_.blocks_mutex_, std::adopt_lock);

      // We collect the slow-path stats only once, since they are not collected
      // per bucket (we pick index 0 arbitrarily). These are also all the host
      // allocations, not taking into account caching and free lists.
      if (i == 0) {
        stats.allocations = stats_.allocations;
        stats.allocated_bytes = stats_.allocated_bytes;
        stats.num_host_alloc = stats.allocations.allocated;
        stats.num_host_free = stats.allocations.freed;
      }

```
- EN: Focus symbols: `size`, `lock`, `gf`, `gb`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`size`, `lock`, `gf`, `gb`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 523-546
```cpp
      // Bucket stats need to be merged with the slow-path stats. We do this in
      // a best effort manner, since we can't really replay the cached events per bucket.
      add_bucket_stats(stats.active_requests, stats_.active_bucket_stats[i]);
      add_bucket_stats(stats.active_bytes, stats_.active_bytes_bucket_stats[i]);
      stats.bucket_allocation[i] = stats_.allocation_bucket_stats[i].allocated;
    }

    // Get the timing stats
    {
      std::lock_guard<std::mutex> g(stats_.timing_mutex_);

      stats.host_alloc_time = stats_.host_alloc_time;
      stats.host_free_time = stats_.host_free_time;
    }

    return stats;
  }

  void resetAccumulatedStats() {
    // Resetting accumulated memory stats requires concurrently holding both the
    // free list mutexes and the blocks mutex. Previously, this was only done in
    // empty_cache function.
    for (size_t i = 0; i < default_pool_.free_list_.size(); ++i) {
      std::lock(
```
- EN: Focus symbols: `add_bucket_stats`, `g`, `resetAccumulatedStats`, `size`, `lock`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`add_bucket_stats`, `g`, `resetAccumulatedStats`, `size`, `lock`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 547-570
```cpp
          default_pool_.free_list_[i].mutex_, default_pool_.blocks_mutex_);
      std::lock_guard<std::mutex> gf(
          default_pool_.free_list_[i].mutex_, std::adopt_lock);
      std::lock_guard<std::mutex> gb(
          default_pool_.blocks_mutex_, std::adopt_lock);

      if (i == 0) {
        stats_.allocations.reset_accumulated();
        stats_.allocated_bytes.reset_accumulated();
      }
      stats_.active_bucket_stats[i].reset_accumulated();
      stats_.active_bytes_bucket_stats[i].reset_accumulated();
      stats_.allocation_bucket_stats[i].reset_accumulated();
      stats_.allocated_bytes_bucket_stats[i].reset_accumulated();
    }

    // Also reset timing stats
    {
      std::lock_guard<std::mutex> g(stats_.timing_mutex_);
      stats_.host_alloc_time.reset_accumulated();
      stats_.host_free_time.reset_accumulated();
    }
  }

```
- EN: Focus symbols: `gf`, `gb`, `reset_accumulated`, `g`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`gf`, `gb`, `reset_accumulated`, `g`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 571-594
```cpp
  void resetPeakStats() {
    // Resetting peak memory stats requires concurrently holding both the
    // free list mutexes and the blocks mutex. Previously, this was only done in
    // empty_cache function.
    for (size_t i = 0; i < default_pool_.free_list_.size(); ++i) {
      std::lock(
          default_pool_.free_list_[i].mutex_, default_pool_.blocks_mutex_);
      std::lock_guard<std::mutex> gf(
          default_pool_.free_list_[i].mutex_, std::adopt_lock);
      std::lock_guard<std::mutex> gb(
          default_pool_.blocks_mutex_, std::adopt_lock);

      if (i == 0) {
        stats_.allocations.reset_peak();
        stats_.allocated_bytes.reset_peak();
      }
      stats_.active_bucket_stats[i].reset_peak();
      stats_.active_bytes_bucket_stats[i].reset_peak();
      stats_.allocation_bucket_stats[i].reset_peak();
      stats_.allocated_bytes_bucket_stats[i].reset_peak();
    }

    // Also reset timing stats
    {
```
- EN: Focus symbols: `resetPeakStats`, `size`, `lock`, `gf`, `gb`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`resetPeakStats`, `size`, `lock`, `gf`, `gb`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 595-622
```cpp
      std::lock_guard<std::mutex> g(stats_.timing_mutex_);
      stats_.host_alloc_time.reset_peak();
      stats_.host_free_time.reset_peak();
    }
  }

 private:
  virtual void add_allocated_block(B* block, BlockPool& pool) {
    std::lock_guard<std::mutex> g(pool.blocks_mutex_);
    pool.blocks_.insert(block);
    stats_.allocations.increase(1);
    stats_.allocated_bytes.increase(block->size_);
    pool.ptr_to_block_.insert({block->ptr_, block});

    // Unfortunately, we have to, on the slow path, quickly
    // lock the bucket to record the allocation. This should
    // be a rare event once the cache is warmed up.
    auto size = block->size_;
    auto index = size_index(size);
    {
      std::lock_guard<std::mutex> g(pool.free_list_[index].mutex_);
      stats_.allocation_bucket_stats[index].increase(1);
      stats_.allocated_bytes_bucket_stats[index].increase(size);
      stats_.active_bucket_stats[index].increase(1);
      stats_.active_bytes_bucket_stats[index].increase(size);
    }
  }

```
- EN: Focus symbols: `g`, `reset_peak`, `add_allocated_block`, `insert`, `increase`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`g`, `reset_peak`, `add_allocated_block`, `insert`, `increase`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 623-646
```cpp
  virtual B* get_free_block(size_t size, BlockPool& pool) {
    auto index = size_index(size);
    std::lock_guard<std::mutex> g(pool.free_list_[index].mutex_);
    if (!pool.free_list_[index].list_.empty()) {
      B* block = pool.free_list_[index].list_.back();
      pool.free_list_[index].list_.pop_back();
      block->allocated_ = true;
      stats_.active_bucket_stats[index].increase(1);
      stats_.active_bytes_bucket_stats[index].increase(size);
      return block;
    }
    return nullptr;
  }

  virtual void process_events(BlockPool& pool) {
    // process all events in the given pool until the last unready event.
    process_events_for_specific_size(-1, pool);
  }

  // If size is -1, process all events from backwards until the last unready
  // event. Otherwise, process events for a specific size and on first ready block
  // is found, add it to the free list and return.
  virtual void process_events_for_specific_size(int64_t size, BlockPool& pool) {
    size_t event_count = 0;
```
- EN: Focus symbols: `get_free_block`, `size_index`, `g`, `empty`, `back`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`get_free_block`, `size_index`, `g`, `empty`, `back`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 647-670
```cpp
    size_t max_events = 0;
    {
      std::lock_guard<std::mutex> g(pool.events_mutex_);
      max_events = pool.events_.size();
    }

    while (true) {
      // Avoid calling cudaEventDestroy while holding a mutex, so move
      // intermediate events out of the lock into this object.
      // process the last event
      std::optional<std::pair<E, B*>> processed;
      {
        std::lock_guard<std::mutex> g(pool.events_mutex_);
        if (!pool.events_.empty()) {
          processed = std::move(pool.events_.back());
          pool.events_.pop_back();
        }
      }

      if (!processed) {
        return;
      }

      if (size != -1) {
```
- EN: Focus symbols: `g`, `size`, `empty`, `move`, `back`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`g`, `size`, `empty`, `move`, `back`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 671-694
```cpp
        if (event_count++ > max_events) {
          {
            std::lock_guard<std::mutex> g(pool.events_mutex_);
            pool.events_.push_front(std::move(*processed));
          }
          return;
        }
        if (size != (int64_t)processed->second->size_) {
          // if we are processing a specific size, and the size of the block
          // doesn't match, we can't use it.
          {
            std::lock_guard<std::mutex> g(pool.events_mutex_);
            pool.events_.push_front(std::move(*processed));
          }
          continue;
        }
      }

      // otherwise, query the event
      {
        // now, see if we can handle this element
        auto& event = processed->first;
        if (!query_event(event)) {
          // push the event onto the back if it's not ready.
```
- EN: Focus symbols: `g`, `push_front`, `move`, `query_event`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`g`, `push_front`, `move`, `query_event`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 695-720
```cpp
          {
            std::lock_guard<std::mutex> g(pool.events_mutex_);
            if (size == -1) {
              pool.events_.push_back(std::move(*processed));
              return;
            } else {
              pool.events_.push_front(std::move(*processed));
              continue;
            }
          }
        }
      }

      // Process the events.
      TORCH_INTERNAL_ASSERT(processed);
      auto* block = processed->second;
      bool available = false;
      {
        std::lock_guard<std::mutex> g(block->mutex_);
        TORCH_INTERNAL_ASSERT(!block->allocated_);
        block->event_count_--;
        if (block->event_count_ == 0) {
          available = true;
        }
      }

```
- EN: Focus symbols: `g`, `push_back`, `move`, `push_front`, `TORCH_INTERNAL_ASSERT`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`g`, `push_back`, `move`, `push_front`, `TORCH_INTERNAL_ASSERT`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 721-744
```cpp
      if (available) {
        auto index = size_index(block->size_);
        std::lock_guard<std::mutex> g(pool.free_list_[index].mutex_);
        pool.free_list_[index].list_.push_back(block);
        stats_.active_bucket_stats[index].decrease(1);
        stats_.active_bytes_bucket_stats[index].decrease(size);
        if (size != -1) {
          return;
        }
      }
    }
  }

  TaskThreadPool* getBackgroundThreadPool() {
    static TaskThreadPool* pool = new TaskThreadPool(1);
    return pool;
  }

 public:
  void begin_allocate_to_pool(
      c10::MempoolId_t pool_id,
      std::function<bool(c10::Stream)> filter) {
    std::unique_lock<std::shared_mutex> lg(instance_mutex_);
    create_or_incref_pool_under_lock(pool_id);
```
- EN: Focus symbols: `size_index`, `g`, `push_back`, `decrease`, `getBackgroundThreadPool`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`size_index`, `g`, `push_back`, `decrease`, `getBackgroundThreadPool`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 745-768
```cpp
    for (auto it2 = captures_underway_.begin(); it2 != captures_underway_.end();
         ++it2) {
      TORCH_CHECK(
          it2->first != pool_id,
          "beginAllocateToPool: already recording to mempool_id");
    }
    captures_underway_.emplace_back(pool_id, std::move(filter));
    captures_underway_empty_.store(false, std::memory_order_relaxed);
  }

  void end_allocate_to_pool(c10::MempoolId_t pool_id) {
    std::unique_lock<std::shared_mutex> lg(instance_mutex_);
    for (auto it = captures_underway_.begin(); it != captures_underway_.end();
         ++it) {
      if (it->first == pool_id) {
        captures_underway_.erase(it);
        captures_underway_empty_.store(captures_underway_.empty(), std::memory_order_relaxed);
        return;
      }
    }
    TORCH_CHECK(false, "endAllocatePool: not currently recording to mempool_id");
  }

  void create_or_incref_pool_under_lock(c10::MempoolId_t pool_id) {
```
- EN: Focus symbols: `begin`, `end`, `TORCH_CHECK`, `emplace_back`, `move`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`begin`, `end`, `TORCH_CHECK`, `emplace_back`, `move`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 769-795
```cpp
    auto it = graph_pools_.find(pool_id);
    if (it == graph_pools_.end()) {
      graph_pools_.emplace(pool_id, std::make_unique<PrivatePool>(pool_id));
    } else {
      TORCH_INTERNAL_ASSERT(it->second->use_count > 0);
      it->second->use_count++;
    }
  }

  void release_pool(c10::MempoolId_t pool_id) {
    std::unique_lock<std::shared_mutex> lg(instance_mutex_);
    auto* pp = graph_pools_.at(pool_id).get();
    TORCH_INTERNAL_ASSERT(pp != nullptr);
    auto uc = --(pp->use_count);
    TORCH_INTERNAL_ASSERT(uc >= 0);
    if (uc == 0) {
      bool inserted = graph_pools_freeable_.insert({pool_id, pp}).second;
      TORCH_INTERNAL_ASSERT(inserted);
    }
  }

 private:
  std::tuple<c10::MempoolId_t, BlockPool&> get_allocation_pool_for_current_stream() {
    if (C10_LIKELY(captures_underway_empty_.load(std::memory_order_relaxed))) {
      return {c10::MempoolId_t{0, 0}, default_pool_};
    }

```
- EN: Focus symbols: `find`, `end`, `emplace`, `TORCH_INTERNAL_ASSERT`, `release_pool`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`find`, `end`, `emplace`, `TORCH_INTERNAL_ASSERT`, `release_pool`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 796-819
```cpp
    std::shared_lock<std::shared_mutex> lg(instance_mutex_);
    S stream = get_current_stream();
    for (auto& entry : captures_underway_) {
      if (entry.second(stream)) {
        auto it = graph_pools_.find(entry.first);
        TORCH_INTERNAL_ASSERT(it != graph_pools_.end());
        auto id = entry.first;
        if (C10_UNLIKELY(!stream_is_capturing(stream))) {
          TORCH_WARN_ONCE("CachingHostAllocator is allocating to private pool (",
                          id.first, ",", id.second,
                          ") but the current stream is not capturing. Private "
                          "pools have been tested only during graph capture. "
                          "See https://github.com/pytorch/pytorch/pull/167507#discussion_r2561698011");
        }
        return {id, it->second->blocks};
      }
    }
    return {c10::MempoolId_t{0, 0}, default_pool_};
  }

  // Helper: return the pool containing a block, based on its owning_pool_.
  BlockPool& pool_from_block(B* block) {
    auto id = block->owning_pool_;
    if (id == c10::MempoolId_t{0, 0}) {
```
- EN: Focus symbols: `lg`, `get_current_stream`, `second`, `find`, `TORCH_INTERNAL_ASSERT`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`lg`, `get_current_stream`, `second`, `find`, `TORCH_INTERNAL_ASSERT`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 820-843
```cpp
      return default_pool_;
    }
    std::shared_lock<std::shared_mutex> lg(instance_mutex_);
    auto it = graph_pools_.find(id);
    TORCH_INTERNAL_ASSERT(it != graph_pools_.end());
    return it->second->blocks;
  }

  B* get_block_from_ptr(void *ptr) {
    std::shared_lock<std::shared_mutex> lk(instance_mutex_);
    {
      std::lock_guard<std::mutex> lk(default_pool_.blocks_mutex_);
      if (default_pool_.ptr_to_block_.count(ptr)) {
        return default_pool_.ptr_to_block_.at(ptr);
      }
    }
    if (C10_LIKELY(graph_pools_.empty())) {
      return nullptr;
    } else {
      for (auto &&[_, private_pool]: graph_pools_) {
        BlockPool& pool = private_pool->blocks;
        std::lock_guard<std::mutex> lk(pool.blocks_mutex_);
        if (pool.ptr_to_block_.count(ptr)) {
          return pool.ptr_to_block_.at(ptr);
```
- EN: Focus symbols: `lg`, `find`, `TORCH_INTERNAL_ASSERT`, `end`, `get_block_from_ptr`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`lg`, `find`, `TORCH_INTERNAL_ASSERT`, `end`, `get_block_from_ptr`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 844-867
```cpp
        }
      }
      return nullptr;
    }
  }

  bool block_exists(void *block_) {
    B *block = reinterpret_cast<B*>(block_);
    std::shared_lock<std::shared_mutex> lk(instance_mutex_);
    {
      std::lock_guard<std::mutex> lk(default_pool_.blocks_mutex_);
      if (default_pool_.blocks_.count(block)) {
        return true;
      }
    }
    if (C10_LIKELY(graph_pools_.empty())) {
      return false;
    } else {
      for (auto &&[_, private_pool]: graph_pools_) {
        BlockPool& pool = private_pool->blocks;
        std::lock_guard<std::mutex> lk(pool.blocks_mutex_);
        if (pool.blocks_.count(block)) {
          return true;
        }
```
- EN: Focus symbols: `block_exists`, `lk`, `count`, `C10_LIKELY`, `empty`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`block_exists`, `lk`, `count`, `C10_LIKELY`, `empty`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 868-892
```cpp
      }
      return false;
    }
  }

protected:
  // We want to keep the non stream capture case as fast as possible,
  // since memory allocation is often in the critical path in non
  // stream capture code. This function will skip the overhead of
  // locking on instance_mutex_, two virtual function calls, and a
  // call into the CUDA API in order to prevent overheads whenever
  // possible.
  bool current_stream_is_capturing_fast_path() const {
    // Stream capture can allocate only to private pools. If there
    // are no private pools for which capture is currently underway,
    // then by modus tollens the current stream is not capturing.
    if (C10_LIKELY(captures_underway_empty_.load(std::memory_order_relaxed))) {
      return false;
    }
    return stream_is_capturing(get_current_stream());
  }

private:
  /* These following functions are runtime-related. */

```
- EN: Focus symbols: `current_stream_is_capturing_fast_path`, `C10_LIKELY`, `load`, `stream_is_capturing`, `get_current_stream`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`current_stream_is_capturing_fast_path`, `C10_LIKELY`, `load`, `stream_is_capturing`, `get_current_stream`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 893-917
```cpp
  // Allocate page-locked memory on the host.
  virtual void allocate_host_memory(size_t size, void** ptr) {
    TORCH_CHECK_NOT_IMPLEMENTED(
        false, "Not implemented for allocate_host_memory");
  }

  // Free block and release the pointer contained in block.
  virtual void free_block(B* block) {
    TORCH_CHECK_NOT_IMPLEMENTED(false, "Not implemented for free_block");
  }

  // Record an event on stream and store event into events.
  virtual void record_stream(std::optional<std::vector<E>>& events, S stream) {
    TORCH_CHECK_NOT_IMPLEMENTED(false, "Not implemented for record_stream");
  }

  // Query event if it is completed.
  virtual bool query_event(E& event) {
    TORCH_CHECK_NOT_IMPLEMENTED(false, "Not implemented for query_event");
  }

  virtual S get_current_stream() const {
    TORCH_CHECK_NOT_IMPLEMENTED(false, "Not implemented for get_current_stream");
  }

```
- EN: Focus symbols: `allocate_host_memory`, `TORCH_CHECK_NOT_IMPLEMENTED`, `free_block`, `record_stream`, `query_event`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`allocate_host_memory`, `TORCH_CHECK_NOT_IMPLEMENTED`, `free_block`, `record_stream`, `query_event`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 918-942
```cpp
  virtual bool stream_is_capturing(S s) const {
    TORCH_CHECK_NOT_IMPLEMENTED(false, "Not implemented for stream_is_capturing");
  }

  // instance variables

  // instance_mutex_ protects graphs_pools_, graph_pools_freeable_,
  // and captures_underway_, as well as the use_count field of
  // PrivatePools in graph_pools_ and graph_pools_freeable_.  We use a
  // shared mutex because we want to allow for multiple private pools
  // to be allocated to concurrently.  Does not protect default_pool_,
  // which has its own mutex.
  alignas(hardware_destructive_interference_size) mutable std::shared_mutex instance_mutex_;

  // We manually maintain the invariant that captures_underway_empty_
  // == captures_underway_.empty() outside of zones guarded by
  // instance_mutex_. This trick allows for us to check whether any
  // captures are currently underway without ever taking a lock on
  // instance_mutex_ (which guards captures_underway_), which is much
  // more expensive than a relaxed memory load on this atomic. Read
  // more here:
  // https://github.com/pytorch/pytorch/pull/167507#discussion_r2586418965
  // It is important to use only "relaxed" loads and stores.
  std::atomic<bool> captures_underway_empty_{true};

```
- EN: Focus symbols: `stream_is_capturing`, `TORCH_CHECK_NOT_IMPLEMENTED`, `alignas`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`stream_is_capturing`, `TORCH_CHECK_NOT_IMPLEMENTED`, `alignas`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 943-969
```cpp
  // Private pools for captures
  ska::flat_hash_map<c10::MempoolId_t, std::unique_ptr<PrivatePool>, c10::MempoolIdHash>
      graph_pools_;

  ska::flat_hash_map<c10::MempoolId_t, PrivatePool*, c10::MempoolIdHash>
      graph_pools_freeable_;

  // Track active capture contexts requesting allocations to specific pools
  std::vector<
      std::pair<c10::MempoolId_t, std::function<bool(c10::Stream)>>> captures_underway_;

  // corresponds to c10::MempoolId_t{0,0}
  BlockPool default_pool_;

  // Indicates whether the event-processing thread pool is active.
  // Set to false in the destructor to signal background threads to stop.
  std::atomic<bool> active_{false};
protected:
  alignas(hardware_destructive_interference_size) HostStatsStaged stats_;
};

struct TORCH_API HostAllocator : public at::Allocator {
  // Associates the pinned memory allocation with a stream to track
  // dependencies. This ensures the memory won't be reused until the stream's
  // operations complete
  virtual bool record_event(void* ptr, void* ctx, c10::Stream stream) = 0;

```
- EN: Focus symbols: `HostAllocator`, `bool`, `alignas`, `record_event`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`HostAllocator`, `bool`, `alignas`, `record_event`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 970-993
```cpp
  // Frees all cached pinned memory and returns it to the system, clearing the
  // allocator's internal cache
  virtual void empty_cache() = 0;

  // Returns comprehensive statistics about the allocator's memory usage,
  // allocation patterns, and timing metrics
  virtual HostStats get_stats() = 0;

  // Resets the cumulative allocation statistics
  virtual void reset_accumulated_stats() = 0;

  // Resets the peak memory usage metrics
  virtual void reset_peak_stats() = 0;

  virtual void begin_allocate_to_pool(
      c10::MempoolId_t pool_id,
      std::function<bool(c10::Stream)> filter) {
    TORCH_CHECK_NOT_IMPLEMENTED(false, "Not implemented for begin_allocate_to_pool");
  }

  virtual void end_allocate_to_pool(c10::MempoolId_t pool_id) {
    TORCH_CHECK_NOT_IMPLEMENTED(false, "Not implemented for end_allocate_to_pool");
  }

```
- EN: Focus symbols: `empty_cache`, `get_stats`, `reset_accumulated_stats`, `reset_peak_stats`, `begin_allocate_to_pool`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`empty_cache`, `get_stats`, `reset_accumulated_stats`, `reset_peak_stats`, `begin_allocate_to_pool`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 994-1019
```cpp
  virtual void release_pool(c10::MempoolId_t pool_id) {
    TORCH_CHECK_NOT_IMPLEMENTED(false, "Not implemented for release_pool");
  }
};

template <typename T, c10::DeleterFnPtr deleteFunc>
struct CachingHostAllocatorInterface : public HostAllocator {
  CachingHostAllocatorInterface() : impl_(std::make_unique<T>()) {}

  at::DataPtr allocate(size_t size) override {
    auto ptr_and_ctx = impl_->allocate(size);
    return {
        ptr_and_ctx.first,
        ptr_and_ctx.second,
        deleteFunc, // Use the template parameter deleter function
        at::DeviceType::CPU};
  }

  void free(void* ctx) {
    impl_->free(ctx);
  }

  bool record_event(void* ptr, void* ctx, c10::Stream stream) override {
    return impl_->record_event(ptr, ctx, stream);
  }

```
- EN: Focus symbols: `CachingHostAllocatorInterface`, `release_pool`, `TORCH_CHECK_NOT_IMPLEMENTED`, `impl_`, `allocate`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`CachingHostAllocatorInterface`, `release_pool`, `TORCH_CHECK_NOT_IMPLEMENTED`, `impl_`, `allocate`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 1020-1046
```cpp
  void empty_cache() override {
    impl_->empty_cache();
  }

  void copy_data(void* dest, const void* src, std::size_t count)
      const override {
    impl_->copy_data(dest, src, count);
  }

  HostStats get_stats() override {
    return impl_->getStats();
  }

  void reset_accumulated_stats() override {
    impl_->resetAccumulatedStats();
  }

  void reset_peak_stats() override {
    impl_->resetPeakStats();
  }

  void begin_allocate_to_pool(
      c10::MempoolId_t pool_id,
      std::function<bool(c10::Stream)> filter) override {
    impl_->begin_allocate_to_pool(pool_id, std::move(filter));
  }

```
- EN: Focus symbols: `empty_cache`, `copy_data`, `get_stats`, `getStats`, `reset_accumulated_stats`. This block implements thin forwarding methods around a lower-level backend object. This keeps the public surface small while centralizing real work in a specialized implementation.
- CN: 关注符号：`empty_cache`, `copy_data`, `get_stats`, `getStats`, `reset_accumulated_stats`。该代码块围绕更底层后端对象实现轻量转发方法。这样可以保持公开接口简洁，同时把真正的工作集中到专门实现中。

### Lines 1047-1070
```cpp
  void end_allocate_to_pool(c10::MempoolId_t pool_id) override {
    impl_->end_allocate_to_pool(pool_id);
  }

  void release_pool(c10::MempoolId_t pool_id) override {
    impl_->release_pool(pool_id);
  }

  std::unique_ptr<T> impl_;
};

#define DECLARE_HOST_ALLOCATOR(name, impl, deleter, instance)       \
  void deleter(void* ptr);                                          \
  struct name final                                                 \
      : public at::CachingHostAllocatorInterface<impl, deleter> {}; \
  static name instance;                                                    \
  void deleter(void* ptr) {                                         \
    instance.free(ptr);                                             \
  }

/**
 * Set the host allocator for DeviceType `device_type`. This allocator manages
 * pinned memory on the host that can be accessed efficiently by the specified
 * device type. Note that this function is not thread-safe.
```
- EN: Focus symbols: `name`, `DECLARE_HOST_ALLOCATOR`, `end_allocate_to_pool`, `release_pool`, `deleter`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`name`, `DECLARE_HOST_ALLOCATOR`, `end_allocate_to_pool`, `release_pool`, `deleter`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 1071-1093
```cpp
 */
TORCH_API void setHostAllocator(
    at::DeviceType device_type,
    at::HostAllocator* allocator,
    uint8_t priority = 0);

TORCH_API at::HostAllocator* getHostAllocator(at::DeviceType device_type);

template <DeviceType device_type>
struct HostAllocatorRegistry {
  explicit HostAllocatorRegistry(HostAllocator* allocator) {
    at::setHostAllocator(device_type, allocator);
  }
};

#define REGISTER_HOST_ALLOCATOR(device_type, allocator) \
  namespace {                                           \
  static at::HostAllocatorRegistry<device_type>         \
      g_host_allocator_registry_instance(allocator);    \
  }

} // namespace at
C10_DIAGNOSTIC_POP()
```
- EN: Focus symbols: `HostAllocatorRegistry`, `REGISTER_HOST_ALLOCATOR`, `at`, `setHostAllocator`, `getHostAllocator`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`HostAllocatorRegistry`, `REGISTER_HOST_ALLOCATOR`, `at`, `setHostAllocator`, `getHostAllocator`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

## Key Concepts / 关键概念
- core ATen/C10 abstractions / ATen/C10 核心抽象
- C++ templates and specialization / C++ 模板与特化
- Type-system design / 类型系统设计
- Runtime validation / 运行时校验
- Backend/device dispatch / 后端/设备分发
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `c10/core/Allocator.h`, `c10/core/AllocatorConfig.h`, `c10/core/Stream.h`, `c10/core/thread_pool.h`, `c10/util/flat_hash_map.h`, `c10/util/llvmMathExtras.h`
- External/system includes / 外部或系统头: `iostream`, `optional`, `deque`, `vector`, `mutex`, `shared_mutex`
- Companion translation units / 配套实现或声明文件: `aten/src/ATen/core/CachingHostAllocator.cpp`
- Inferred semantic dependencies / 推断出的语义依赖: device dispatch / 设备分发; C++ templates / C++ 模板; runtime validation / 运行时校验; namespace scoping / 命名空间作用域
