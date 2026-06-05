# CUDACachingAllocator.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `c10/cuda/CUDACachingAllocator.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements memory-allocation interfaces, policies, and configuration shared by c10 runtime components.
- **Purpose (CN)**: 实现 c10 运行时组件共享的内存分配接口、策略与配置。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21
```cpp
#pragma once

#include <c10/core/AllocatorConfig.h>
#include <c10/core/CachingDeviceAllocator.h>
#include <c10/cuda/CUDAAllocatorConfig.h>
#include <c10/cuda/CUDAGraphsC10Utils.h>
#include <c10/cuda/CUDAMacros.h>
#include <c10/cuda/CUDAStream.h>
#include <c10/util/Exception.h>
#include <c10/util/Registry.h>

#include <atomic>
#include <cstddef>
#include <cstdint>
#include <functional>
#include <memory>
#include <string>
#include <unordered_set>
#include <utility>

namespace c10 {
```
- **EN**: This block assembles the compilation dependencies, pulling in local PyTorch/c10 headers such as c10/core/AllocatorConfig.h, c10/core/CachingDeviceAllocator.h, c10/cuda/CUDAAllocatorConfig.h, and 5 more; standard-library headers such as atomic, cstddef, cstdint, and 5 more. The preprocessor guard keeps declarations single-instanced when this header is included transitively. The namespace declarations place the code inside c10, matching the surrounding subsystem. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently.
- **CN**: 这一段组织编译依赖，引入了本地 PyTorch/c10 头文件，如 c10/core/AllocatorConfig.h、c10/core/CachingDeviceAllocator.h、c10/cuda/CUDAAllocatorConfig.h 等共 8 项；标准库头文件，如 atomic、cstddef、cstdint 等共 8 项。 预处理器保护用于避免头文件在传递包含时被重复展开。 命名空间声明把代码放入 c10 中，与周边子系统保持一致。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。

### Lines 23-46
```cpp
// Caching allocator will execute every registered callback if it unable to find
// block inside of already allocated area.
class C10_CUDA_API FreeMemoryCallback {
 public:
  virtual ~FreeMemoryCallback() = default;
  virtual bool Execute() = 0;
};

C10_DECLARE_REGISTRY(FreeCudaMemoryCallbacksRegistry, FreeMemoryCallback);
#define REGISTER_FREE_MEMORY_CALLBACK(name, ...) \
  C10_REGISTER_CLASS(FreeCudaMemoryCallbacksRegistry, name, __VA_ARGS__)
} // namespace c10
  //
// TODO: Turn this into an honest to goodness class. I briefly attempted to do
// this, but it was a bit irritating to figure out how to also correctly
// apply pimpl pattern so I didn't have to leak any internal implementation
// details in the header (CUDACachingAllocator could be made a pimpl, but
// you also need to appropriately define a class which is a subclass
// of Allocator. Not impossible, but required a bit more surgery than
// I wanted to do at the time.)
//
// Why is this using a namespace rather than old-style THCCachingAllocator_
// prefix?  Mostly because it made the HIPify rules easier to write; _ is
// not counted as a word boundary, so you would otherwise have to list each
```
- **EN**: The preprocessor guard keeps declarations single-instanced when this header is included transitively. It introduces or extends C10_CUDA_API, which, a, which define the main data structures or interfaces for this portion of the file. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 它引入或扩展了 C10_CUDA_API、which、a，这些类型定义了本段涉及的主要数据结构或接口。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 47-65
```cpp
// of these functions.

namespace c10::cuda::CUDACachingAllocator {

// Preserved only for BC reasons
// NOLINTNEXTLINE(misc-unused-using-decls)
using c10::CachingDeviceAllocator::AllocatorTraceTracker;
using c10::CachingDeviceAllocator::BlockInfo;
using c10::CachingDeviceAllocator::CreateContextFn;
using c10::CachingDeviceAllocator::DeviceStats;
using c10::CachingDeviceAllocator::RecordContext;
using c10::CachingDeviceAllocator::SegmentInfo;
using c10::CachingDeviceAllocator::TraceEntry;

struct AllocatorState {
  virtual ~AllocatorState() = default;
};

struct AllocatorConfigInfo {
```
- **EN**: The namespace declarations place the code inside c10::cuda::CUDACachingAllocator, matching the surrounding subsystem. It introduces or extends c10, c10, c10, and 6 more, which define the main data structures or interfaces for this portion of the file. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically.
- **CN**: 命名空间声明把代码放入 c10::cuda::CUDACachingAllocator 中，与周边子系统保持一致。 它引入或扩展了 c10、c10、c10 等共 9 项，这些类型定义了本段涉及的主要数据结构或接口。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。

### Lines 66-87
```cpp
  double garbage_collection_threshold;
  size_t max_split_size;
  size_t pinned_num_register_threads;
  bool expandable_segments;
  bool release_lock_on_malloc;
  bool pinned_use_host_register;
  bool graph_capture_record_stream_reuse;
  std::string last_allocator_settings;
  std::vector<size_t> roundup_power2_divisions;
};

struct SnapshotInfo {
  std::vector<CachingDeviceAllocator::SegmentInfo> segments;
  std::vector<std::vector<CachingDeviceAllocator::TraceEntry>> device_traces;
  std::vector<CachingDeviceAllocator::AnnotationEntry> external_annotations;
  AllocatorConfigInfo config_metadata;
};

// returns the pointers freed in the pool
// and the pointers allocated. Note: a pointer
// may appear in both freed and allocated
struct CheckpointDelta {
```
- **EN**: It introduces or extends SnapshotInfo, CheckpointDelta, which define the main data structures or interfaces for this portion of the file. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently.
- **CN**: 它引入或扩展了 SnapshotInfo、CheckpointDelta，这些类型定义了本段涉及的主要数据结构或接口。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。

### Lines 88-110
```cpp
  std::vector<void*> ptrs_freed;
  std::vector<at::DataPtr> dataptrs_allocd;
};

using OutOfMemoryObserver = std::function<void(
    int64_t device,
    size_t allocated,
    size_t device_total,
    size_t device_free)>;

// Observer called when an allocation is preemptively rejected due to
// throw_on_cudamalloc_oom policy. Parameters:
//   - device: GPU device index
//   - alloc_size: size of the rejected allocation request
//   - total_allocated: total memory allocated before the request
//   - device_total: total GPU memory
using OomRejectionObserver = std::function<void(
    int64_t device,
    size_t alloc_size,
    size_t total_allocated,
    size_t device_total)>;

struct ShareableHandle {
```
- **EN**: It introduces or extends OutOfMemoryObserver, OomRejectionObserver, ShareableHandle, which define the main data structures or interfaces for this portion of the file. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently.
- **CN**: 它引入或扩展了 OutOfMemoryObserver、OomRejectionObserver、ShareableHandle，这些类型定义了本段涉及的主要数据结构或接口。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。

### Lines 111-134
```cpp
  ptrdiff_t offset;
  std::string handle;
};

struct StreamSegmentSize {
  StreamSegmentSize(cudaStream_t s, bool small_, size_t sz)
      : stream(s), is_small_pool(small_), total_size(sz) {}
  cudaStream_t stream;
  bool is_small_pool;
  size_t total_size;
};

class CUDAAllocator : public DeviceAllocator {
 public:
  virtual void* raw_alloc(size_t nbytes) = 0;
  virtual void* raw_alloc_with_stream(size_t nbytes, cudaStream_t stream) = 0;
  virtual void raw_delete(void* ptr) = 0;
  virtual void init(int device_count) = 0;
  virtual double getMemoryFraction(c10::DeviceIndex device) = 0;
  virtual void setMemoryFraction(double fraction, c10::DeviceIndex device) = 0;
  virtual std::vector<StreamSegmentSize> getExpandableSegmentSizes(
      c10::DeviceIndex device) = 0;
  virtual void enable(bool value) = 0;
  virtual bool isEnabled() const = 0;
```
- **EN**: It introduces or extends StreamSegmentSize, CUDAAllocator, which define the main data structures or interfaces for this portion of the file. This chunk defines `StreamSegmentSize`, which manages device or stream context while preserving execution invariants. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently.
- **CN**: 它引入或扩展了 StreamSegmentSize、CUDAAllocator，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `StreamSegmentSize`，其作用是管理设备或流上下文，同时保持执行不变量。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。

### Lines 135-158
```cpp
  virtual void cacheInfo(c10::DeviceIndex device, size_t* largestBlock) = 0;
  virtual void* getBaseAllocation(void* ptr, size_t* size) = 0;
  // Keep for BC only
  virtual void recordStream(const DataPtr& ptr, CUDAStream stream) = 0;
  void recordStream(const DataPtr& ptr, c10::Stream stream) override {
    CUDAStream cuda_stream = CUDAStream(stream);
    recordStream(ptr, cuda_stream);
  }
  virtual SnapshotInfo snapshot(
      MempoolId_t mempool_id = {0, 0},
      bool include_traces = true) = 0;
  virtual void beginAllocateToPool(
      c10::DeviceIndex device,
      MempoolId_t mempool_id,
      std::function<bool(cudaStream_t)> filter) = 0;
  virtual void endAllocateToPool(
      c10::DeviceIndex device,
      MempoolId_t mempool_id) = 0;
  virtual void releasePool(c10::DeviceIndex device, MempoolId_t mempool_id) = 0;
  virtual int getPoolUseCount(
      c10::DeviceIndex /*device*/,
      MempoolId_t /*mempool_id*/) {
    TORCH_CHECK(
        false,
```
- **EN**: This chunk defines `getPoolUseCount`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `getPoolUseCount`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 159-182
```cpp
        name(),
        " does not yet support getPoolUseCount. "
        "If you need it, please file an issue describing your use case.");
  }
  virtual void createOrIncrefPool(
      c10::DeviceIndex /*device*/,
      MempoolId_t /*mempool_id*/,
      std::shared_ptr<CUDAAllocator> allocator = nullptr) {
    TORCH_CHECK(
        false,
        name(),
        " does not yet support createOrIncrefPool. "
        "If you need it, please file an issue describing your use case.");
  }
  virtual void setUseOnOOM(
      c10::DeviceIndex device,
      MempoolId_t mempool_id,
      bool use_on_oom) {
    TORCH_CHECK(
        false,
        name(),
        " does not yet support setUseOnOOM. "
        "If you need it, please file an issue describing your use case.");
  }
```
- **EN**: This chunk defines `setUseOnOOM`, which maintains lookup structures and hashing behavior for fast metadata access. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Reference-count operations preserve strong/weak ownership invariants for shared runtime objects. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `setUseOnOOM`，其作用是维护查找结构与哈希行为，以便快速访问元数据。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 引用计数操作用于维护共享运行时对象的强/弱所有权不变量。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 183-201
```cpp
  virtual void setNoSplit(c10::DeviceIndex device, MempoolId_t mempool_id) {
    TORCH_CHECK(
        false,
        name(),
        " does not yet support setNoSplit. "
        "If you need it, please file an issue describing your use case.");
  }

  // returns true if the allocated blocks are equal to expected live allocations
  virtual bool checkPoolLiveAllocations(
      c10::DeviceIndex /*device*/,
      MempoolId_t /*mempool_id*/,
      const std::unordered_set<void*>& /*expected_live_allocations*/) {
    TORCH_CHECK(
        false,
        name(),
        " does not yet support checkPoolLiveAllocations. "
        "If you need it, please file an issue describing your use case.");
  }
```
- **EN**: This chunk defines `checkPoolLiveAllocations`, which manages allocation, reuse, or release decisions for runtime memory. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `checkPoolLiveAllocations`，其作用是管理运行时内存的分配、复用或释放决策。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 202-225
```cpp
  virtual ShareableHandle shareIpcHandle(void* ptr) = 0;
  virtual std::shared_ptr<void> getIpcDevPtr(std::string handle) = 0;
  virtual bool isHistoryEnabled() {
    TORCH_CHECK(
        false,
        name(),
        " does not yet support recordHistory. "
        "If you need it, please file an issue describing your use case.");
  }
  virtual std::shared_ptr<GatheredContext> getContextForPointer(
      const void* ptr) {
    return nullptr;
  }
  virtual void recordHistory(
      bool enabled,
      CreateContextFn context_recorder,
      size_t alloc_trace_max_entries,
      RecordContext when,
      bool clearHistory,
      const std::vector<std::string>& skip_actions) = 0;
  virtual void recordAnnotation(
      const std::vector<std::pair<std::string, std::string>>& /*md*/) {}
  virtual void pushCompileContext(std::string& md) {}
  virtual void popCompileContext() {}
```
- **EN**: This chunk defines `popCompileContext`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `popCompileContext`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 226-243
```cpp
  virtual void setUserMetadata(const std::string& metadata) {}
  virtual std::string getUserMetadata() {
    return "";
  }
  virtual void attachOutOfMemoryObserver(OutOfMemoryObserver observer) = 0;
  virtual void attachOomRejectionObserver(OomRejectionObserver observer) = 0;

  // Attached AllocatorTraceTracker callbacks will be called while the
  // per-device allocator lock is held. Any additional locks taken from within
  // the callback must be proven to always have the lock order that never
  // triggers a deadlock. In particular, Python's GIL may be held when
  // calling the allocator so it is unsafe to try to acquire the GIL in this
  // callback.
  virtual void attachAllocatorTraceTracker(AllocatorTraceTracker tracker) = 0;

  virtual void enablePeerAccess(
      c10::DeviceIndex dev,
      c10::DeviceIndex dev_to_access) = 0;
```
- **EN**: This chunk defines `getUserMetadata`, which implements a focused piece of backend/runtime support logic. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `getUserMetadata`，其作用是实现一段聚焦的后端/运行时支持逻辑。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 245-268
```cpp
  // memory not allocated from cudaMalloc cannot be copied
  // across devices using cudaMemcpyAsync if peer to peer access is disabled.
  // instead it requires cudaMemcpyAsyncPeer
  //  with P2P Enabled, all combinations work
  //  with P2P Disabled:
  //                       cudaMalloc cudaMallocAsync/cuMemMap
  // cudaMemcpyAsyncPeer   works      works
  // cudaMemcpyAsync       works      error

  // This function performs chooses to use the Peer version of
  // memcpy if required based on where the allocated put dst/src.
  virtual cudaError_t memcpyAsync(
      void* dst,
      int dstDevice,
      const void* src,
      int srcDevice,
      size_t count,
      cudaStream_t stream,
      bool p2p_enabled) = 0;
  virtual std::shared_ptr<AllocatorState> getCheckpointState(
      c10::DeviceIndex device,
      MempoolId_t id) = 0;
  virtual CheckpointDelta setCheckpointPoolState(
      c10::DeviceIndex device,
```
- **EN**: It introduces or extends cudaMemcpyAsync, which define the main data structures or interfaces for this portion of the file. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 它引入或扩展了 cudaMemcpyAsync，这些类型定义了本段涉及的主要数据结构或接口。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 269-289
```cpp
      std::shared_ptr<AllocatorState> pps) = 0;
  virtual std::string name() = 0;
  std::pair<size_t, size_t> getMemoryInfo(c10::DeviceIndex device) override {
    c10::DeviceGuard device_guard({at::kCUDA, device});
    size_t free = 0;
    size_t total = 0;
    C10_CUDA_CHECK(cudaMemGetInfo(&free, &total));
    return {free, total};
  }
};

// Allocator object, statically initialized
// See BackendInitializer in CUDACachingAllocator.cpp.
// Atomic loads on x86 are just normal loads,
// (atomic stores are different), so reading this value
// is no different than loading a pointer.
C10_CUDA_API extern std::atomic<CUDAAllocator*> allocator;

inline CUDAAllocator* get() {
  return allocator.load();
}
```
- **EN**: This chunk defines `load`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `load`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 291-314
```cpp
// Called directly by clients.
inline void* raw_alloc(size_t nbytes) {
  return get()->raw_alloc(nbytes);
}

inline void* raw_alloc_with_stream(size_t nbytes, cudaStream_t stream) {
  return get()->raw_alloc_with_stream(nbytes, stream);
}

inline void raw_delete(void* ptr) {
  get()->raw_delete(ptr);
}

inline void init(int device_count) {
  get()->init(device_count);
}

inline double getMemoryFraction(c10::DeviceIndex device) {
  return get()->getMemoryFraction(device);
}

inline void setMemoryFraction(double fraction, c10::DeviceIndex device) {
  get()->setMemoryFraction(fraction, device);
}
```
- **EN**: This chunk defines `setMemoryFraction`, which maintains lookup structures and hashing behavior for fast metadata access. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `setMemoryFraction`，其作用是维护查找结构与哈希行为，以便快速访问元数据。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 316-339
```cpp
inline std::vector<StreamSegmentSize> getExpandableSegmentSizes(
    c10::DeviceIndex device) {
  return get()->getExpandableSegmentSizes(device);
}

inline void emptyCache(MempoolId_t mempool_id = {0, 0}) {
  get()->emptyCache(mempool_id);
}

inline void enable(bool value) {
  get()->enable(value);
}

inline bool isEnabled() {
  return get()->isEnabled();
}

inline void cacheInfo(c10::DeviceIndex device, size_t* largestBlock) {
  get()->cacheInfo(device, largestBlock);
}

inline void* getBaseAllocation(void* ptr, size_t* size) {
  return get()->getBaseAllocation(ptr, size);
}
```
- **EN**: This chunk defines `getBaseAllocation`, which manages allocation, reuse, or release decisions for runtime memory. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `getBaseAllocation`，其作用是管理运行时内存的分配、复用或释放决策。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 341-362
```cpp
inline void recordStream(const DataPtr& dataPtr, CUDAStream stream) {
  get()->recordStream(dataPtr, stream);
}

inline c10::CachingDeviceAllocator::DeviceStats getDeviceStats(
    c10::DeviceIndex device) {
  return get()->getDeviceStats(device);
}

inline void resetAccumulatedStats(c10::DeviceIndex device) {
  get()->resetAccumulatedStats(device);
}

inline void resetPeakStats(c10::DeviceIndex device) {
  get()->resetPeakStats(device);
}

inline SnapshotInfo snapshot(
    MempoolId_t mempool_id = {0, 0},
    bool include_traces = true) {
  return get()->snapshot(mempool_id, include_traces);
}
```
- **EN**: This chunk defines `resetPeakStats`, which maintains lookup structures and hashing behavior for fast metadata access. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `resetPeakStats`，其作用是维护查找结构与哈希行为，以便快速访问元数据。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 364-386
```cpp
inline std::shared_ptr<AllocatorState> getCheckpointState(
    c10::DeviceIndex device,
    MempoolId_t id) {
  return get()->getCheckpointState(device, id);
}

inline CheckpointDelta setCheckpointPoolState(
    c10::DeviceIndex device,
    std::shared_ptr<AllocatorState> pps) {
  return get()->setCheckpointPoolState(device, std::move(pps));
}

// CUDAGraph interactions
inline void beginAllocateToPool(
    c10::DeviceIndex device,
    MempoolId_t mempool_id,
    std::function<bool(cudaStream_t)> filter) {
  get()->beginAllocateToPool(device, mempool_id, std::move(filter));
}

inline void endAllocateToPool(c10::DeviceIndex device, MempoolId_t mempool_id) {
  get()->endAllocateToPool(device, mempool_id);
}
```
- **EN**: This chunk defines `endAllocateToPool`, which manages allocation, reuse, or release decisions for runtime memory. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `endAllocateToPool`，其作用是管理运行时内存的分配、复用或释放决策。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 388-411
```cpp
inline void recordHistory(
    bool enabled,
    CreateContextFn context_recorder,
    size_t alloc_trace_max_entries,
    RecordContext when,
    bool clearHistory,
    const std::vector<std::string>& skip_actions) {
  get()->recordHistory(
      enabled,
      context_recorder,
      alloc_trace_max_entries,
      when,
      clearHistory,
      skip_actions);
}

inline void recordAnnotation(
    const std::vector<std::pair<std::string, std::string>>& md) {
  get()->recordAnnotation(md);
}

inline void pushCompileContext(std::string& md) {
  get()->pushCompileContext(md);
}
```
- **EN**: This chunk defines `pushCompileContext`, which implements a focused piece of backend/runtime support logic. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently.
- **CN**: 这一段定义了 `pushCompileContext`，其作用是实现一段聚焦的后端/运行时支持逻辑。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。

### Lines 413-435
```cpp
inline void popCompileContext() {
  get()->popCompileContext();
}

inline bool isHistoryEnabled() {
  return get()->isHistoryEnabled();
}

inline std::shared_ptr<GatheredContext> getContextForPointer(const void* ptr) {
  return get()->getContextForPointer(ptr);
}

inline bool checkPoolLiveAllocations(
    c10::DeviceIndex device,
    MempoolId_t mempool_id,
    const std::unordered_set<void*>& expected_live_allocations) {
  return get()->checkPoolLiveAllocations(
      device, mempool_id, expected_live_allocations);
}

inline void attachOutOfMemoryObserver(OutOfMemoryObserver observer) {
  get()->attachOutOfMemoryObserver(std::move(observer));
}
```
- **EN**: This chunk defines `attachOutOfMemoryObserver`, which converts one representation into another form used by nearby runtime code. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `attachOutOfMemoryObserver`，其作用是把一种表示转换为附近运行时代码使用的另一种形式。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 437-459
```cpp
inline void attachOomRejectionObserver(OomRejectionObserver observer) {
  get()->attachOomRejectionObserver(std::move(observer));
}

inline void attachAllocatorTraceTracker(AllocatorTraceTracker tracker) {
  get()->attachAllocatorTraceTracker(std::move(tracker));
}

inline void releasePool(c10::DeviceIndex device, MempoolId_t mempool_id) {
  get()->releasePool(device, mempool_id);
}
inline void createOrIncrefPool(
    c10::DeviceIndex device,
    MempoolId_t mempool_id,
    std::shared_ptr<CUDAAllocator> allocator_ptr = nullptr) {
  get()->createOrIncrefPool(device, mempool_id, std::move(allocator_ptr));
}
inline void setUseOnOOM(
    c10::DeviceIndex device,
    MempoolId_t mempool_id,
    bool use_on_oom) {
  get()->setUseOnOOM(device, mempool_id, use_on_oom);
}
```
- **EN**: This chunk defines `setUseOnOOM`, which maintains lookup structures and hashing behavior for fast metadata access. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Reference-count operations preserve strong/weak ownership invariants for shared runtime objects. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently.
- **CN**: 这一段定义了 `setUseOnOOM`，其作用是维护查找结构与哈希行为，以便快速访问元数据。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 引用计数操作用于维护共享运行时对象的强/弱所有权不变量。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。

### Lines 460-478
```cpp
inline void setNoSplit(c10::DeviceIndex device, MempoolId_t mempool_id) {
  get()->setNoSplit(device, mempool_id);
}
inline int getPoolUseCount(c10::DeviceIndex device, MempoolId_t mempool_id) {
  return get()->getPoolUseCount(device, mempool_id);
}

// Not part of CUDA_ALLOCATOR_BACKEND_INTERFACE
inline std::shared_ptr<void> getIpcDevPtr(std::string handle) {
  return get()->getIpcDevPtr(std::move(handle));
}

inline ShareableHandle shareIpcHandle(void* ptr) {
  return get()->shareIpcHandle(ptr);
}

inline std::string name() {
  return get()->name();
}
```
- **EN**: This chunk defines `name`, which implements a focused piece of backend/runtime support logic. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `name`，其作用是实现一段聚焦的后端/运行时支持逻辑。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 480-500
```cpp
inline cudaError_t memcpyAsync(
    void* dst,
    int dstDevice,
    const void* src,
    int srcDevice,
    size_t count,
    cudaStream_t stream,
    bool p2p_enabled) {
  return get()->memcpyAsync(
      dst, dstDevice, src, srcDevice, count, stream, p2p_enabled);
}

inline void enablePeerAccess(
    c10::DeviceIndex dev,
    c10::DeviceIndex dev_to_access) {
  get()->enablePeerAccess(dev, dev_to_access);
}

inline void setUserMetadata(const std::string& metadata) {
  get()->setUserMetadata(metadata);
}
```
- **EN**: This chunk defines `setUserMetadata`, which maintains lookup structures and hashing behavior for fast metadata access. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `setUserMetadata`，其作用是维护查找结构与哈希行为，以便快速访问元数据。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 502-512
```cpp
inline std::string getUserMetadata() {
  return get()->getUserMetadata();
}

} // namespace c10::cuda::CUDACachingAllocator

namespace c10::cuda {
// Keep BC only
using c10::CaptureId_t;
using c10::MempoolId_t;
} // namespace c10::cuda
```
- **EN**: The namespace declarations place the code inside c10::cuda, matching the surrounding subsystem. It introduces or extends c10, c10, which define the main data structures or interfaces for this portion of the file. This chunk defines `get`, which implements a focused piece of backend/runtime support logic. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 命名空间声明把代码放入 c10::cuda 中，与周边子系统保持一致。 它引入或扩展了 c10、c10，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `get`，其作用是实现一段聚焦的后端/运行时支持逻辑。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 末尾会返回累积结果，或把更新后的状态交还给调用方。


## Key Concepts / 关键概念
- **CUDA runtime support**
  - EN: Implements CUDA-facing c10 runtime support, including allocators, streams, guards, and device helpers.
  - CN: 实现面向 CUDA 的 c10 运行时支持，包括分配器、流、守卫以及设备辅助逻辑。
- **C10_CUDA_API**
  - EN: `C10_CUDA_API` is one of the dominant symbols declared or implemented in this file.
  - CN: `C10_CUDA_API` 是本文件声明或实现的关键符号之一。
- **which**
  - EN: `which` is one of the dominant symbols declared or implemented in this file.
  - CN: `which` 是本文件声明或实现的关键符号之一。
- **Memory allocation**
  - EN: Tracks how raw memory is requested, cached, released, and reported.
  - CN: 跟踪原始内存如何被申请、缓存、释放以及上报。
- **Device abstraction**
  - EN: Represents backend/device identity so code can stay portable across runtimes.
  - CN: 表示后端/设备身份，使代码在不同运行时之间保持可移植。
- **Stream semantics**
  - EN: Coordinates asynchronous execution ordering and per-stream resource usage.
  - CN: 协调异步执行顺序以及按流划分的资源使用。

## Dependencies / 依赖关系
- **Internal includes / 内部依赖**: `c10/core/AllocatorConfig.h`、`c10/core/CachingDeviceAllocator.h`、`c10/cuda/CUDAAllocatorConfig.h`、`c10/cuda/CUDAGraphsC10Utils.h`、`c10/cuda/CUDAMacros.h`、`c10/cuda/CUDAStream.h`、`c10/util/Exception.h`、`c10/util/Registry.h`
- **Third-party includes / 第三方依赖**: 无
- **Standard includes / 标准库依赖**: `atomic`、`cstddef`、`cstdint`、`functional`、`memory`、`string`、`unordered_set`、`utility`
- **System includes / 系统依赖**: 无
- **Namespaces / 命名空间**: `c10`、`c10::cuda::CUDACachingAllocator`、`c10::cuda`
- **Representative symbols / 代表性符号**: `C10_CUDA_API`、`which`、`a`、`c10`、`AllocatorState`、`AllocatorConfigInfo`、`SnapshotInfo`、`CheckpointDelta`、`OutOfMemoryObserver`、`OomRejectionObserver`
