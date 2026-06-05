# CUDAMallocAsyncAllocator.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `c10/cuda/CUDAMallocAsyncAllocator.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements memory-allocation interfaces, policies, and configuration shared by c10 runtime components.
- **Purpose (CN)**: 实现 c10 运行时组件共享的内存分配接口、策略与配置。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17
```cpp
#include <c10/cuda/CUDACachingAllocator.h>
#include <c10/cuda/CUDAException.h>
#include <c10/cuda/CUDAFunctions.h>
#include <c10/cuda/CUDAGuard.h>
#include <c10/util/UniqueVoidPtr.h>
#include <c10/util/flat_hash_map.h>

#include <unordered_set>
#include <vector>

namespace c10::cuda::CUDACachingAllocator::CudaMallocAsync {

using namespace c10::CachingAllocator;
using namespace c10::CachingDeviceAllocator;

// CUDA device allocator that uses cudaMallocAsync to implement
// the same interface as CUDACachingAllocator.cpp.
```
- **EN**: This block assembles the compilation dependencies, pulling in local PyTorch/c10 headers such as c10/cuda/CUDACachingAllocator.h, c10/cuda/CUDAException.h, c10/cuda/CUDAFunctions.h, and 3 more; standard-library headers such as unordered_set, vector. The namespace declarations place the code inside c10::cuda::CUDACachingAllocator::CudaMallocAsync, matching the surrounding subsystem. It introduces or extends namespace, namespace, which define the main data structures or interfaces for this portion of the file. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently.
- **CN**: 这一段组织编译依赖，引入了本地 PyTorch/c10 头文件，如 c10/cuda/CUDACachingAllocator.h、c10/cuda/CUDAException.h、c10/cuda/CUDAFunctions.h 等共 6 项；标准库头文件，如 unordered_set、vector。 命名空间声明把代码放入 c10::cuda::CUDACachingAllocator::CudaMallocAsync 中，与周边子系统保持一致。 它引入或扩展了 namespace、namespace，这些类型定义了本段涉及的主要数据结构或接口。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。

### Lines 19-41
```cpp
// Designed to be safe for CUDA graph capture.
// Interactions with CUDA graph capture are mediated by
// notifyCaptureBegin
// notifyCaptureAboutToEnd
// notifyCaptureEnded
// notifyCaptureDestroy

// Implementation details, not declared in CUDACachingAllocator.h
namespace {

// General helpers

struct UsageStream {
  cudaStream_t stream;
  c10::DeviceIndex device;
  UsageStream() = default;
  UsageStream(cudaStream_t s, c10::DeviceIndex d) : stream(s), device(d) {}
  UsageStream(const UsageStream& us) = default;
  UsageStream(UsageStream&& us) noexcept = default;
  UsageStream& operator=(const UsageStream& other) = default;
  UsageStream& operator=(UsageStream&& other) noexcept = default;
  ~UsageStream() = default;
};
```
- **EN**: It introduces or extends UsageStream, which define the main data structures or interfaces for this portion of the file. This chunk defines `UsageStream`, which manages device or stream context while preserving execution invariants. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically.
- **CN**: 它引入或扩展了 UsageStream，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `UsageStream`，其作用是管理设备或流上下文，同时保持执行不变量。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。

### Lines 43-61
```cpp
bool operator==(const UsageStream& lhs, const UsageStream& rhs) {
  return (lhs.stream == rhs.stream) && (lhs.device == rhs.device);
}

struct UsageStreamHash {
  size_t operator()(const UsageStream& us) const noexcept {
    return std::hash<void*>{}(us.stream) + static_cast<size_t>(us.device);
  }
};

struct PtrUsage {
  // recorded_streams holds side usage streams added by record_stream calls.
  // In other words, it does NOT include the original creation stream.
  ska::flat_hash_set<UsageStream, UsageStreamHash> recorded_streams;
  UsageStream creation_stream{};
  uint64_t size;
  bool captured;
  PtrUsage(uint64_t s, bool c) : size(s), captured(c) {}
};
```
- **EN**: It introduces or extends UsageStreamHash, PtrUsage, which define the main data structures or interfaces for this portion of the file. This chunk defines `PtrUsage`, which implements a focused piece of backend/runtime support logic. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 它引入或扩展了 UsageStreamHash、PtrUsage，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `PtrUsage`，其作用是实现一段聚焦的后端/运行时支持逻辑。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 63-86
```cpp
int device_count = 0;
// these don't need to be c10::once_flags as in CUDAGeneratorImpl.cpp
// because they'll only be flipped by functions that have locked the mutex.
std::vector<bool> devs_initialized_flags;
std::vector<UsageStream> dummy_unifying_free_streams;

// Possible micro-optimization:
// Some accesses to ptr_info are read-only.
// We could let those be concurrent with a shared_mutex and
// have concurrent calls take a shared_lock.
// Keeping it simple with an ordinary mutex for now.
std::mutex general_mutex;

/**
 * Note [Avoid freeing uncaptured ptrs during CUDA graph capture]
 * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 * During CUDA graph capture, it's illegal to call cudaFreeAsync
 * on a pointer that came from a non-captured cudaMallocAsync.
 * Unfortunately, Python being what it is, it's impossible to be
 * sure no uncaptured tensor will ever have its destructor called
 * in a capturing region.
 * We avoid errors by
 *  1. remembering if allocated pointers were captured or uncaptured
 *  2. during capture, if we detect an attempt to free an uncaptured
```
- **EN**: This chunk continues `PtrUsage` and expands its control flow, data movement, or edge-case handling. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 这一段延续了 `PtrUsage`，进一步展开其控制流、数据流转或边界处理逻辑。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 87-102
```cpp
 *     allocation on a capturing stream, don't free it immediately,
 *     just remember it and defer its cudaFreeAsync call to after
 *     the end of capture (specifically, to notifyCaptureEnded).
 */

using PtrInfo = ska::flat_hash_map<void*, PtrUsage>;
PtrInfo ptr_info;
std::vector<void*> ungraphed_ptrs_defer_free_until_no_capture;

// These two help setMemoryFraction limit the amount of memory
// used by PyTorch in particular (as opposed to other libraries
// in the same process that might be sharing the same cudaMemPool_t).
std::vector<size_t> pytorch_used_bytes;
std::vector<size_t> pytorch_memory_limits;

// Graph-specific helpers
```
- **EN**: It introduces or extends PtrInfo, which define the main data structures or interfaces for this portion of the file. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently.
- **CN**: 它引入或扩展了 PtrInfo，这些类型定义了本段涉及的主要数据结构或接口。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。

### Lines 104-127
```cpp
/**
 * Note [Avoid dangling free streams during CUDA graph capture]
 * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 * During capture, all stream dependencies must branch out from
 * the stream on which capture began and rejoin this initial stream
 * before capture ends.
 * The user rigs desired forking and joining with event waits.
 * But it's hard to be sure when tensor destructors get called relative
 * to the final joins.
 * For example, suppose a user
 *   forks work stream B from initial capture stream A
 *   creates a tensor T in B
 *   joins by syncing A with B
 *   ends capture.
 * All well and good, right? Maybe not: maybe T went out of scope
 * and its destructor got called AFTER the rejoin, leaving the graph with
 * "unjoined work": a dangling cudaFreeAsync node in stream B.
 * Ensuring that all tensor destructors for all side stream tensors
 * are called before side streams rejoin the main stream is
 * difficult. The user might have to add a bunch of explicit
 * "del"s at the right spots in code that was fine for ordinary
 * eager execution.
 * Fortunately, we can spare the user this burden:
 * during capture, we remember _all_ free streams,
```
- **EN**: Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically.
- **CN**: 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。

### Lines 128-147
```cpp
 * and manually rejoin them with the capture stream during
 * notifyCaptureAboutToEnd.
 * This approach is heavy-handed, but hopefully capture only needs to
 * happen once, so we don't mind being heavy-handed.
 *
 * TODO: If, someday, we augment the graph bindings to support recapture
 * https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#whole-graph-update
 * (eg, as a way to accommodate dynamic params) we should think more
 * carefully about the CPU overhead of remembering and rejoining
 * all free streams during capture. Maybe it's not a big deal.
 */
std::unordered_set<UsageStream, UsageStreamHash> capture_free_streams;
bool capture_underway = false;

// Implementation functions

// Assumes the caller holds general_mutex
inline void lazy_init_device(c10::DeviceIndex device) {
  if (!devs_initialized_flags[device]) {
    CUDAGuard g(device);
```
- **EN**: This chunk defines `g`, which implements a focused piece of backend/runtime support logic. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 这一段定义了 `g`，其作用是实现一段聚焦的后端/运行时支持逻辑。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 149-171
```cpp
    // See "Retaining memory in the pool" here:
    // https://developer.nvidia.com/blog/using-cuda-stream-ordered-memory-allocator-part-1/
    cudaMemPool_t mempool = nullptr;
    C10_CUDA_CHECK(cudaDeviceGetDefaultMemPool(&mempool, device));
    uint64_t threshold = UINT64_MAX;
    C10_CUDA_CHECK(cudaMemPoolSetAttribute(
        mempool, cudaMemPoolAttrReleaseThreshold, &threshold));

    // I think all these are on by default, but I want to enable them
    // explicitly to ensure awareness.
    int enable = 1;
    C10_CUDA_CHECK(cudaMemPoolSetAttribute(
        mempool, cudaMemPoolReuseFollowEventDependencies, &enable));
    C10_CUDA_CHECK(cudaMemPoolSetAttribute(
        mempool, cudaMemPoolReuseAllowOpportunistic, &enable));
    C10_CUDA_CHECK(cudaMemPoolSetAttribute(
        mempool, cudaMemPoolReuseAllowInternalDependencies, &enable));

    // Grabs a stream from the current device to use as the "unifier" free
    // stream for allocations that end up used on multiple streams.
    const auto dufs = getStreamFromPool();
    dummy_unifying_free_streams[device] =
        UsageStream(dufs.stream(), dufs.device_index());
```
- **EN**: This chunk declares `UsageStream`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段声明了 `UsageStream`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 173-194
```cpp
    pytorch_used_bytes[device] = 0;
    pytorch_memory_limits[device] = UINT64_MAX;

    devs_initialized_flags[device] = true;
  }
}

inline void sync_raw(cudaStream_t dependency, cudaStream_t dependent) {
  // CUDACachingAllocator.cpp uses raw cuda events, as do we.
  cudaEvent_t event = nullptr;
  C10_CUDA_CHECK(cudaEventCreateWithFlags(&event, cudaEventDisableTiming));
  C10_CUDA_CHECK(cudaEventRecord(event, dependency));
  C10_CUDA_CHECK(cudaStreamWaitEvent(dependent, event));
  C10_CUDA_CHECK(cudaEventDestroy(event));
}

// Assumes the caller holds general_mutex
inline void free_impl(PtrInfo::iterator& it) {
  // Possible micro-optimization: If we did a value-copy here, we could move
  // ptr_info.erase(it) up here and drop the lock immediately.
  const auto& recorded_streams = it->second.recorded_streams;
  const auto& creation_stream = it->second.creation_stream;
```
- **EN**: This chunk defines `free_impl`, which manages allocation, reuse, or release decisions for runtime memory. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `free_impl`，其作用是管理运行时内存的分配、复用或释放决策。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 196-211
```cpp
  // If the usage stream is a null (default) stream,
  // cudaFreeAsync infers the device from the ambient context,
  // so we need to set the right ambient context.
  CUDAGuard g(creation_stream.device);

  if (recorded_streams.empty()) {
    // ptr was only used on one stream, which must have been
    // the original allocation stream.
    // Frees ptr in the original allocation stream.

    C10_CUDA_CHECK(cudaFreeAsync(it->first, creation_stream.stream));

    if (C10_UNLIKELY(capture_underway)) {
      // See Note [Avoid dangling free streams during CUDA graph capture]
      capture_free_streams.insert(creation_stream);
    }
```
- **EN**: This chunk defines `insert`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `insert`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 212-229
```cpp
  } else {
    // ptr was used on many streams. We don't know which was the most recent.
    // There could even have been multiple most recent usage streams acting
    // on different regions of the memory.
    // But cudaFreeAsync only accepts a single most recent usage stream.
    // We can still safely free ptr with a trick:
    // Use a dummy "unifying stream", sync the unifying stream with all of
    // ptr's usage streams, and pass the dummy stream to cudaFreeAsync.

    // Retrieves the dummy "unifier" stream from the device
    // on which the pointer was originally allocated.
    auto dummy_unifying_free_stream =
        dummy_unifying_free_streams[creation_stream.device];
    TORCH_INTERNAL_ASSERT(
        dummy_unifying_free_stream.device == creation_stream.device);

    // we're already on creation_stream.device, no need to re-guard
    sync_raw(creation_stream.stream, dummy_unifying_free_stream.stream);
```
- **EN**: This chunk defines `sync_raw`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `sync_raw`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 231-254
```cpp
    // The number of usage streams is typically small (low single digits)
    for (const auto& recorded_stream : recorded_streams) {
      // Logic here accommodates the chance some of the usage streams were on
      // other devices, which is possible if some usage kernels accessed the
      // memory via p2p.

      // cudaEventRecord requires that the input event and stream are on the
      // same device.
      CUDAGuard g_usage(recorded_stream.device);

      sync_raw(recorded_stream.stream, dummy_unifying_free_stream.stream);
    }

    // Frees ptr in the dummy "unifier" stream.
    C10_CUDA_CHECK(cudaFreeAsync(it->first, dummy_unifying_free_stream.stream));
    // At this point, unless dummy_unifying_free_stream happens to alias some
    // future user stream, the allocation is only available for "opportunistic"
    // reuse, ie, if the CPU sees dummy_unifying_free_stream has reached the
    // point that all events recorded on all usage streams have resolved from
    // the CPU's perspective. In theory, we could remove the need for the driver
    // to do this tracking by e.g. replacing
    // cudaStreamWaitEvent(dummy_unifying_free_stream.stream, event);
    // with
    // cudaStreamWaitEvent(creation_stream.stream, event);
```
- **EN**: This chunk defines `cudaStreamWaitEvent`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `cudaStreamWaitEvent`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 255-277
```cpp
    // then cudaFreeAsyncing straight back into creation_stream.stream,
    // but this forces a potentially false dependency of creation_stream.stream
    // on all the recorded_streams.

    if (C10_UNLIKELY(capture_underway)) {
      // See Note [Avoid dangling free streams during CUDA graph capture]
      capture_free_streams.emplace(
          dummy_unifying_free_stream.stream, dummy_unifying_free_stream.device);
    }
  }

  pytorch_used_bytes[creation_stream.device] -= it->second.size;

  ptr_info.erase(it);
}

void freeAsync(void* ptr) {
  std::lock_guard<std::mutex> lk(general_mutex);

  auto err = cudaGetLastError();
  C10_CUDA_CHECK(err);
  auto it = ptr_info.find(ptr);
  TORCH_INTERNAL_ASSERT(it != ptr_info.end(), "ptr not found in ptr_info");
```
- **EN**: This chunk defines `find`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `find`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 279-299
```cpp
  if (C10_UNLIKELY(capture_underway)) {
    if (!it->second.captured) {
      TORCH_WARN_ONCE(
          "freeAsync() was called on an uncaptured allocation during graph capture "
          "(address = ",
          ptr,
          "). This may be benign, for example, a Python tensor in the capture "
          "might happen to shadow (use the same name as) an unrelated temporary "
          "tensor from somewhere before capture, pushing the earlier tensor "
          "out of scope. "
          "However, if the tensor we're freeing here IS used by the capture, "
          "freeing it is an error, and may cause illegal memory accesses or "
          "memory corruption during graph replay.");
      // See Note [Avoid freeing uncaptured ptrs during CUDA graph capture]
      // Remembers the raw pointer, not the iterator.
      // This forces notifyCaptureEnded to do another lookup,
      // but avoids the risk the iterator might be invalidated
      // between now and then.
      ungraphed_ptrs_defer_free_until_no_capture.push_back(ptr);
      return;
    }
```
- **EN**: This chunk defines `push_back`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `push_back`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 300-323
```cpp
  } else if (C10_UNLIKELY(it->second.captured)) {
    TORCH_WARN(
        "Attempting uncaptured free of a captured allocation with address ",
        ptr,
        "\nThis is technically allowed, but may indicate you are losing "
        "the last user-visible tensor through which the allocation can "
        "be accessed, so you'll have no way to view the data after "
        "future replays of the owning graph.");
  }

  free_impl(it);
}

// Symmetric with NativeCachingAllocator::malloc for now,
// although I don't think we absolutely need the symmetry.
void mallocAsync(
    void** devPtr,
    c10::DeviceIndex device,
    size_t size,
    cudaStream_t stream) {
  TORCH_INTERNAL_ASSERT(
      0 <= device && device < device_count,
      "Invalid device index ",
      static_cast<int>(device),
```
- **EN**: This chunk defines `mallocAsync`, which manages allocation, reuse, or release decisions for runtime memory. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `mallocAsync`，其作用是管理运行时内存的分配、复用或释放决策。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 324-345
```cpp
      ": did you call init?");

  // If stream is a null (default) stream,
  // cudaMallocAsync infers the device from the ambient context,
  // so we need to set the right ambient context.
  CUDAGuard g(device);

  std::lock_guard<std::mutex> lk(general_mutex);

  if (!capture_underway &&
      !ungraphed_ptrs_defer_free_until_no_capture.empty()) {
    // See Note [Avoid freeing uncaptured ptrs during CUDA graph capture]
    for (const auto ptr : ungraphed_ptrs_defer_free_until_no_capture) {
      auto it = ptr_info.find(ptr);
      TORCH_INTERNAL_ASSERT(it != ptr_info.end(), "ptr not found in ptr_info");
      free_impl(it);
    }

    ungraphed_ptrs_defer_free_until_no_capture.clear();
  }

  lazy_init_device(device);
```
- **EN**: This chunk defines `lazy_init_device`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `lazy_init_device`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 347-370
```cpp
  // Defensively checks for preexisting CUDA error state.
  auto err = cudaGetLastError();
  C10_CUDA_CHECK(err);

  // TODO: Could we avoid calling cudaMallocAsync while holding general_mutex,
  // perhaps by letting lazy_init_device use separate once_flags or an internal
  // static initializer?
  if (pytorch_used_bytes[device] + size > pytorch_memory_limits[device]) {
    err = cudaErrorMemoryAllocation;
  } else {
    err = cudaMallocAsync(devPtr, size, stream);
  }

  if (err == cudaErrorMemoryAllocation) {
    // Clears CUDA's internal error state so the user, if desired, can catch the
    // OOM exception, free some stuff on the script side, and retry the
    // allocation. This aligns with the behavior of alloc_block in
    // CUDACachingAllocator.cpp.
    (void)cudaGetLastError(); // clear CUDA error
    size_t device_free = 0;
    size_t device_total = 0;
    C10_CUDA_CHECK(cudaMemGetInfo(&device_free, &device_total));
    TORCH_CHECK_WITH(
        OutOfMemoryError,
```
- **EN**: This chunk defines `cudaMallocAsync`, which manages allocation, reuse, or release decisions for runtime memory. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `cudaMallocAsync`，其作用是管理运行时内存的分配、复用或释放决策。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 371-388
```cpp
        false,
        "Allocation on device ",
        static_cast<int>(device),
        " would exceed allowed memory. (out of memory)",
        "\nCurrently allocated     : ",
        format_size(pytorch_used_bytes[device]),
        "\nRequested               : ",
        format_size(size),
        "\nDevice limit            : ",
        format_size(device_total),
        "\nFree (according to CUDA): ",
        format_size(device_free),
        "\nPyTorch limit (set by user-supplied memory fraction)"
        "\n                        : ",
        format_size(pytorch_memory_limits[device]));
  } else {
    C10_CUDA_CHECK(err);
  }
```
- **EN**: This chunk defines `static_cast<int>`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `static_cast<int>`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 390-406
```cpp
  auto inserted = ptr_info.emplace(*devPtr, PtrUsage(size, capture_underway));
  TORCH_INTERNAL_ASSERT(
      inserted.second,
      "address returned by cudaMallocAsync already exists "
      "in ptr_info");

  inserted.first->second.creation_stream = {stream, device};

  pytorch_used_bytes[device] += size;
}

} // anonymous namespace

static void local_raw_delete(void* ptr);

// Same pattern as CUDACachingAllocator.cpp.
struct CudaMallocAsyncAllocator : public CUDAAllocator {
```
- **EN**: It introduces or extends CudaMallocAsyncAllocator, which define the main data structures or interfaces for this portion of the file. This chunk defines `local_raw_delete`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 它引入或扩展了 CudaMallocAsyncAllocator，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `local_raw_delete`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 407-423
```cpp
  DataPtr allocate(size_t size) override {
    constexpr size_t one_exa_bytes = 1152921504606846976ULL;
    TORCH_CHECK_WITH(
        OutOfMemoryError,
        size < one_exa_bytes,
        "CUDA out of memory. Tried to allocate more than 1EB memory.");
    c10::DeviceIndex device = 0;
    C10_CUDA_CHECK(c10::cuda::GetDevice(&device));
    void* r = nullptr;
    if (size != 0) {
      mallocAsync(&r, device, size, cuda::getCurrentCUDAStream(device));
    }
    return {r, r, &local_raw_delete, Device(DeviceType::CUDA, device)};
  }
  DeleterFnPtr raw_deleter() const override {
    return &local_raw_delete;
  }
```
- **EN**: This chunk defines `raw_deleter`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `raw_deleter`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 425-445
```cpp
  // This function should not issue any context-creating calls,
  // just set up for later calls to init per-device pools based
  // on the current device each later call sees.
  void init(int dev_count) override {
    static bool called = [](int dev_count) {
      // Are there external guarantees init will be called before
      // any of the allocator's other functions?
      // std::lock_guard<std::mutex> lk(general_mutex);
      device_count = dev_count;
      devs_initialized_flags.resize(dev_count, false);
      dummy_unifying_free_streams.resize(dev_count);
      pytorch_used_bytes.resize(dev_count);
      pytorch_memory_limits.resize(dev_count);
      return true;
    }(dev_count);
    (void)called;
  }

  bool initialized() override {
    return !devs_initialized_flags.empty();
  }
```
- **EN**: This chunk defines `empty`, which implements a focused piece of backend/runtime support logic. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `empty`，其作用是实现一段聚焦的后端/运行时支持逻辑。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 447-463
```cpp
  static void assertValidDevice(c10::DeviceIndex device) {
    TORCH_CHECK(
        0 <= device && device < device_count, "Invalid device argument.");
  }

  double getMemoryFraction(c10::DeviceIndex device) override {
    std::lock_guard<std::mutex> lk(general_mutex);
    assertValidDevice(device);
    CUDAGuard g(device);
    lazy_init_device(device);

    size_t device_free = 0;
    size_t device_total = 0;
    C10_CUDA_CHECK(cudaMemGetInfo(&device_free, &device_total));
    return static_cast<double>(pytorch_memory_limits[device]) /
        static_cast<double>(device_total);
  }
```
- **EN**: This chunk defines `static_cast<double>`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `static_cast<double>`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 465-485
```cpp
  void setMemoryFraction(double fraction, c10::DeviceIndex device) override {
    TORCH_INTERNAL_ASSERT(
        0 <= fraction && fraction <= 1,
        "invalid fraction:",
        fraction,
        ". Please set within (0, 1).");

    std::lock_guard<std::mutex> lk(general_mutex);
    assertValidDevice(device);
    CUDAGuard g(device);
    // Should setMemoryFraction be allowed to trigger a full device context and
    // pool-creating lazy_init_device, or should we simply assert this device is
    // already initialized, ie
    // TORCH_CHECK(devs_initialized_flags[device], ...)?
    lazy_init_device(device);

    size_t device_free = 0;
    size_t device_total = 0;
    C10_CUDA_CHECK(cudaMemGetInfo(&device_free, &device_total));
    pytorch_memory_limits[device] =
        static_cast<uint64_t>(fraction * static_cast<double>(device_total));
```
- **EN**: This chunk defines `static_cast<uint64_t>`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `static_cast<uint64_t>`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 487-508
```cpp
    // Alternative: Instead of a manual hard limit, we could use
    // cudaMemPoolSetAttribute(mempool, cudaMemPoolAttrReleaseThreshold,
    // &threshold); This is a soft hint: The driver allows the pool's reserved
    // memory to spike above threshold in regions of high cudaMallocAsync
    // demand, but opportunistically trims reserved memory back to threshold
    // when the memory in use is < threshold. I don't like this because it
    // introduces performance nondeterminism.
  }

  std::vector<StreamSegmentSize> getExpandableSegmentSizes(
      c10::DeviceIndex device) override {
    TORCH_CHECK(
        false,
        "CUDAMallocAsyncAllocator does not yet support getExpandableSegmentSizes.");
  }

  void emptyCache(/*unused*/ MempoolId_t mempool_id) override {
    std::lock_guard<std::mutex> lk(general_mutex);

    for (int dev = 0; dev < device_count; dev++) {
      if (devs_initialized_flags[dev]) {
        CUDAGuard g(static_cast<c10::DeviceIndex>(dev));
```
- **EN**: This chunk defines `g`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `g`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 510-524
```cpp
        cudaMemPool_t mempool = nullptr;
        C10_CUDA_CHECK(cudaDeviceGetDefaultMemPool(&mempool, dev));
        C10_CUDA_CHECK(cudaDeviceSynchronize());
        C10_CUDA_CHECK(cudaMemPoolTrimTo(mempool, 0));
      }
    }
  }

  void enable(bool /*value*/) override {
    // cannot disable
  }

  bool isEnabled() const override {
    return true;
  }
```
- **EN**: This chunk defines `isEnabled`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `isEnabled`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 526-549
```cpp
  void cacheInfo(c10::DeviceIndex device, size_t* maxWorkspaceGuess) override {
    // The only consumer of cacheInfo is getMaxWorkspaceSize in Conv_v7.cpp.
    // Afaict, the role of cacheInfo is to give getMaxWorkspaceSize a reasonable
    // maximum workspace size to use for an upcoming cudnnFind call.
    //
    // The native allocator's cacheInfo chooses to return the size of its
    // largest unused block (which is the largest allocation the native
    // allocator can service immediately and asynchronously without a
    // cudaMalloc.
    //
    // Here, we use a different heuristic: figure out the max usable workspace
    // size with a bit of educated trial and error. It's ok to be
    // perf-inefficient because cacheInfo is a prelude to cudnnFind.
    //
    // The algo cache then stores the best-performing algo with workspace <=
    // maxWorkspaceGuess. Later calls with the same param set hit in cache and
    // try to allocate the same workspace. If, in one of those future calls,
    // workspace allocation fails (ie because less ambient memory is available),
    // the bindings rerun cudnnFind, including calling cacheInfo again
    // beforehand to estimate a new (smaller) largest-available workspace. Over
    // a few such calls, the cache should settle to the algo with a workspace
    // size that's small enough to succeed every time (for that param set).
    //
    // So the strategy here is to return a rough, largeish guess and let the
```
- **EN**: This chunk defines `cacheInfo`, which implements a focused piece of backend/runtime support logic. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `cacheInfo`，其作用是实现一段聚焦的后端/运行时支持逻辑。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 550-572
```cpp
    // bindings retry to trim as needed over time.
    //
    // The only caveat is, even if a workspace is allocated without OOM errors
    // now and in future calls, it's hard to be sure those later error-free
    // cudaMallocAsyncs are fast and come straight from the pool (ie,
    // cudaMallocAsync didn't need to reserve more memory from the system).
    // Hopefully, after repeated workspace requests, the pool's reserved memory
    // also stabilizes to a point where they all come straight from the pool.
    std::lock_guard<std::mutex> lk(general_mutex);
    assertValidDevice(device);
    CUDAGuard g(device);
    lazy_init_device(device);

    size_t free_upper_bound = 0;
    size_t device_total = 0;
    C10_CUDA_CHECK(cudaMemGetInfo(&free_upper_bound, &device_total));
    TORCH_INTERNAL_ASSERT(
        free_upper_bound + pytorch_used_bytes[device] <= device_total);
    size_t guess = std::min(
        free_upper_bound,
        pytorch_memory_limits[device] - pytorch_used_bytes[device]);
    auto stream = c10::cuda::getCurrentCUDAStream();
    void* dummy = nullptr;
```
- **EN**: This chunk declares `getCurrentCUDAStream`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段声明了 `getCurrentCUDAStream`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 574-597
```cpp
    // Defensively checks for preexisting CUDA error state.
    auto err = cudaGetLastError();
    C10_CUDA_CHECK(err);

    while (true) {
      // Duplicates some logic from mallocAsync to work with the error state
      // directly instead of repeatedly catching an exception thrown by
      // mallocAsync.
      if (pytorch_used_bytes[device] + guess > pytorch_memory_limits[device]) {
        err = cudaErrorMemoryAllocation;
      } else {
        err = cudaMallocAsync(&dummy, guess, stream);
      }

      if (err == cudaSuccess) {
        C10_CUDA_CHECK(cudaFreeAsync(dummy, stream));
        *maxWorkspaceGuess = guess;
        return;
      } else if (err == cudaErrorMemoryAllocation) {
        (void)cudaGetLastError(); // clear CUDA error
        guess >>= 1; // quick and dirty: try half the size next iteration
      } else {
        C10_CUDA_CHECK(err);
      }
```
- **EN**: This chunk defines `cudaMallocAsync`, which manages allocation, reuse, or release decisions for runtime memory. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `cudaMallocAsync`，其作用是管理运行时内存的分配、复用或释放决策。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 598-621
```cpp
    }
  }

  void* getBaseAllocation(void* ptr, size_t* size) override {
    std::lock_guard<std::mutex> lk(general_mutex);

    auto it = ptr_info.find(ptr);
    TORCH_INTERNAL_ASSERT(it != ptr_info.end(), "ptr not found in ptr_info");

    if (size) {
      *size = it->second.size;
    }

    return ptr;
  }

  void recordStream(const DataPtr& ptr, cuda::CUDAStream stream) override {
    std::lock_guard<std::mutex> lk(general_mutex);
    auto ptr_val = ptr.get();
    // Empty tensor's storage().data() might be a null ptr. As there is no
    // blocks associated with those tensors, it is fine to do nothing here.
    if (!ptr_val) {
      return;
    }
```
- **EN**: This chunk defines `storage`, which converts one representation into another form used by nearby runtime code. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `storage`，其作用是把一种表示转换为附近运行时代码使用的另一种形式。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 623-642
```cpp
    // The pointer should exist in the map already.
    auto it = ptr_info.find(ptr_val);
    TORCH_INTERNAL_ASSERT(it != ptr_info.end(), "ptr not found in ptr_info");

    UsageStream to_record{stream.stream(), stream.device_index()};
    if (to_record == it->second.creation_stream) {
      TORCH_WARN_ONCE(
          "Called record_stream on tensor whose original creation stream "
          "matches the recorded stream. This is unnecessary and has no effect.");
    } else {
      it->second.recorded_streams.insert(to_record);
    }
  }

  ShareableHandle shareIpcHandle(void* handle) override {
    TORCH_CHECK(
        false,
        "cudaMallocAsync does not yet support shareIpcHandle. "
        "If you need it, please file an issue describing your use case.");
  }
```
- **EN**: This chunk defines `shareIpcHandle`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `shareIpcHandle`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 644-662
```cpp
  std::shared_ptr<void> getIpcDevPtr(std::string handle) override {
    TORCH_CHECK(
        false,
        "cudaMallocAsync does not yet support getIpcDevPtr. "
        "If you need it, please file an issue describing your use case.");
  }

  void recordHistory(
      bool enabled,
      CreateContextFn context_recorder,
      size_t alloc_trace_max_entries,
      RecordContext when,
      bool clearHistory,
      const std::vector<std::string>& skip_actions) override {
    TORCH_CHECK(
        false,
        "cudaMallocAsync does not yet support recordHistory. "
        "If you need it, please file an issue describing your use case.");
  }
```
- **EN**: This chunk defines `recordHistory`, which converts one representation into another form used by nearby runtime code. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `recordHistory`，其作用是把一种表示转换为附近运行时代码使用的另一种形式。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 664-683
```cpp
  void attachOutOfMemoryObserver(OutOfMemoryObserver observer) override {
    TORCH_CHECK(
        false,
        "cudaMallocAsync does not yet support attachOutOfMemoryObserver. "
        "If you need it, please file an issue describing your use case.");
  }

  void attachOomRejectionObserver(OomRejectionObserver observer) override {
    TORCH_CHECK(
        false,
        "cudaMallocAsync does not yet support attachOomRejectionObserver. "
        "If you need it, please file an issue describing your use case.");
  }

  void attachAllocatorTraceTracker(AllocatorTraceTracker tracker) override {
    TORCH_CHECK(
        false,
        "cudaMallocAsync does not yet support attachAllocatorTraceTracker. "
        "If you need it, please file an issue describing your use case.");
  }
```
- **EN**: This chunk defines `attachAllocatorTraceTracker`, which manages allocation, reuse, or release decisions for runtime memory. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `attachAllocatorTraceTracker`，其作用是管理运行时内存的分配、复用或释放决策。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 685-706
```cpp
  std::shared_ptr<AllocatorState> getCheckpointState(
      c10::DeviceIndex device,
      MempoolId_t id) override {
    TORCH_CHECK(
        false,
        "cudaMallocAsync does not yet support getCheckpointState. "
        "If you need it, please file an issue describing your use case.");
  }

  CheckpointDelta setCheckpointPoolState(
      c10::DeviceIndex device,
      std::shared_ptr<AllocatorState> pps) override {
    TORCH_CHECK(
        false,
        "cudaMallocAsync does not yet support setCheckpointPoolState. "
        "If you need it, please file an issue describing your use case.");
  }

  // Collects stats for device.
  // If device hasn't been used yet, returns 0s without creating a context.
  DeviceStats getDeviceStats(c10::DeviceIndex device) override {
    assertValidDevice(device);
```
- **EN**: This chunk defines `assertValidDevice`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `assertValidDevice`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 708-728
```cpp
    // Memory currently reserved by the mempool
    uint64_t reserved_mem_current = 0;
    // High-water mark of memory reserved by the mempool since last reset
    uint64_t reserved_mem_peak = 0;
    // Memory currently in use by the mempool
    uint64_t used_mem_current = 0;
    // High-water mark of memory
    uint64_t used_mem_peak = 0;

    std::lock_guard<std::mutex> lk(general_mutex);

    if (devs_initialized_flags[device]) {
      CUDAGuard g(device);

      cudaMemPool_t mempool = nullptr;
      C10_CUDA_CHECK(cudaDeviceGetDefaultMemPool(&mempool, device));
      C10_CUDA_CHECK(cudaMemPoolGetAttribute(
          mempool, cudaMemPoolAttrReservedMemCurrent, &reserved_mem_current));

      C10_CUDA_CHECK(cudaMemPoolGetAttribute(
          mempool, cudaMemPoolAttrReservedMemHigh, &reserved_mem_peak));
```
- **EN**: This chunk defines `g`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `g`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 730-753
```cpp
      C10_CUDA_CHECK(cudaMemPoolGetAttribute(
          mempool, cudaMemPoolAttrUsedMemCurrent, &used_mem_current));

      C10_CUDA_CHECK(cudaMemPoolGetAttribute(
          mempool, cudaMemPoolAttrUsedMemHigh, &used_mem_peak));
    }

    // Many stat types are specific to the native allocator. We leave these
    // untouched. Their "struct Stat"s will contain zeroed values.
    DeviceStats stats;

    // In the native allocator:
    // allocated_bytes is the total bytes of blocks that have been malloc()ed
    // and not yet free()d.
    // active_bytes is the total bytes of blocks that have been malloc()ed but
    // not yet released back into a free pool. In other words, it includes all
    // allocated_bytes, as well as the bytes of "limbo state" blocks had have
    // already been free()ed but not yet free_block()ed back into a pool due to
    // outstanding stream_uses.
    //
    // Here, in the cudaMallocAsync allocator:
    // We simply ask the driver's opinion about active memory.
    // We don't bother distinguishing between allocated_bytes and active_bytes.
    stats.allocated_bytes[static_cast<size_t>(StatType::AGGREGATE)].current =
```
- **EN**: It introduces or extends Stat, which define the main data structures or interfaces for this portion of the file. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 它引入或扩展了 Stat，这些类型定义了本段涉及的主要数据结构或接口。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 754-776
```cpp
        static_cast<int64_t>(used_mem_current);
    stats.allocated_bytes[static_cast<size_t>(StatType::AGGREGATE)].peak =
        static_cast<int64_t>(used_mem_peak);
    stats.active_bytes[static_cast<size_t>(StatType::AGGREGATE)].current =
        static_cast<int64_t>(used_mem_current);
    stats.active_bytes[static_cast<size_t>(StatType::AGGREGATE)].peak =
        static_cast<int64_t>(used_mem_peak);
    stats.reserved_bytes[static_cast<size_t>(StatType::AGGREGATE)].current =
        static_cast<int64_t>(reserved_mem_current);
    stats.reserved_bytes[static_cast<size_t>(StatType::AGGREGATE)].peak =
        static_cast<int64_t>(reserved_mem_peak);

    return stats;
  }

  void resetAccumulatedStats(c10::DeviceIndex device) override {
    assertValidDevice(device);
    TORCH_WARN_ONCE(
        "For backend:cudaMallocAsync, resetAccumulatedStats has no effect.");
  }

  void resetPeakStats(c10::DeviceIndex device) override {
    assertValidDevice(device);
```
- **EN**: This chunk defines `resetPeakStats`, which maintains lookup structures and hashing behavior for fast metadata access. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `resetPeakStats`，其作用是维护查找结构与哈希行为，以便快速访问元数据。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 778-800
```cpp
    CUDAGuard g(device);
    cudaMemPool_t mempool = nullptr;
    C10_CUDA_CHECK(cudaDeviceGetDefaultMemPool(&mempool, device));
    // Using zero as the reset value is the method recommended by Cuda driver
    // team. Vivek Kini says:
    //   "Resetting to zero (which is the only valid value when setting
    //    ReservedMemHigh) resets it to ReservedMemCurrent inside the driver
    //   (same goes for UsedMemHigh/UsedMemCurrent)"
    uint64_t zero = 0;
    C10_CUDA_CHECK(cudaMemPoolSetAttribute(
        mempool, cudaMemPoolAttrReservedMemHigh, &zero));
    C10_CUDA_CHECK(
        cudaMemPoolSetAttribute(mempool, cudaMemPoolAttrUsedMemHigh, &zero));
  }

  SnapshotInfo snapshot(MempoolId_t mempool_id, bool include_traces) override {
    TORCH_CHECK(
        false,
        "Calling snapshot with backend:cudaMallocAsync is not meaningful. "
        "(For backend:native, snapshot returns a detailed summary of all "
        "blocks tracked by the allocator, but the cudaMallocAsync backend "
        "does not track individual blocks.)");
  }
```
- **EN**: This chunk defines `snapshot`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `snapshot`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 802-820
```cpp
  // CUDAGraph interactions
  void beginAllocateToPool(
      c10::DeviceIndex device,
      MempoolId_t mempool_id,
      std::function<bool(cudaStream_t)> /*filter*/) override {
    std::lock_guard<std::mutex> lk(general_mutex);

    TORCH_INTERNAL_ASSERT(capture_free_streams.empty());
    TORCH_CHECK(
        !capture_underway,
        "Only one capture at a time is allowed in a process.")
    capture_underway = true;
  }

  void endAllocateToPool(c10::DeviceIndex device, MempoolId_t mempool_id)
      override {
    assertValidDevice(device);

    std::lock_guard<std::mutex> lk(general_mutex);
```
- **EN**: This chunk defines `assertValidDevice`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `assertValidDevice`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 822-841
```cpp
    TORCH_CHECK(
        capture_underway,
        "CudaMallocAsync::notifyCaptureAboutToEnd called, "
        "but CudaMallocAsync::capture_underway is false.");

    auto capture_stream = cuda::getCurrentCUDAStream(device);

    // See Note [Avoid dangling free streams during CUDA graph capture]
    for (const auto& free_stream : capture_free_streams) {
      // cudaEventRecord requires that the input event and stream are on the
      // same device.
      CUDAGuard g(free_stream.device);

      // CUDACachingAllocator.cpp uses raw cuda events, as do we.
      cudaEvent_t event = nullptr;
      C10_CUDA_CHECK(cudaEventCreateWithFlags(&event, cudaEventDisableTiming));
      C10_CUDA_CHECK(cudaEventRecord(event, free_stream.stream));
      C10_CUDA_CHECK(cudaStreamWaitEvent(capture_stream.stream(), event));
      C10_CUDA_CHECK(cudaEventDestroy(event));
    }
```
- **EN**: This chunk defines `g`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `g`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 843-864
```cpp
    capture_free_streams.clear();
    TORCH_CHECK(
        capture_underway,
        "CudaMallocAsync::notifyCaptureEnded called, "
        "but CudaMallocAsync::capture_underway is false.");
    capture_underway = false;
  }

  void releasePool(c10::DeviceIndex device, MempoolId_t mempool_id) override {
    // Q: Do we need to do anything special here, like clear long-lived
    //    pointers created during the original capture (for example,
    //    tensors intended as the graph's I/O surface) that might still
    //    be resident in ptr_info?
    // A: I don't think so.
    //    Those allocations survived capture because the user held
    //    explicit tensor references to them,
    //    Those tensors' destructors will call freeAsync() on each pointer
    //    when the user is done with them.
    //    The freeAsync()s will probably incur
    //    TORCH_WARN("Attempting uncaptured free of a captured allocation..."
    //    but stale ptrs will not permanently leak into ptr_info.
  }
```
- **EN**: This chunk defines `releasePool`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `releasePool`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 866-889
```cpp
  void* raw_alloc(size_t nbytes) override {
    if (nbytes == 0) {
      return nullptr;
    }
    c10::DeviceIndex device = 0;
    C10_CUDA_CHECK(c10::cuda::GetDevice(&device));
    void* r = nullptr;
    mallocAsync(&r, device, nbytes, cuda::getCurrentCUDAStream(device));
    return r;
  }

  void* raw_alloc_with_stream(size_t nbytes, cudaStream_t stream) override {
    if (nbytes == 0) {
      return nullptr;
    }
    c10::DeviceIndex device = 0;
    C10_CUDA_CHECK(c10::cuda::GetDevice(&device));
    void* r = nullptr;
    mallocAsync(&r, device, nbytes, stream);
    return r;
  }
  void raw_delete(void* ptr) override {
    freeAsync(ptr);
  }
```
- **EN**: This chunk defines `freeAsync`, which manages allocation, reuse, or release decisions for runtime memory. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `freeAsync`，其作用是管理运行时内存的分配、复用或释放决策。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 890-905
```cpp
  void enablePeerAccess(c10::DeviceIndex dev, c10::DeviceIndex dev_to_access)
      override {
    // Double-checks allocator backend hasn't changed, which would definitely be
    // an error. cudaMallocAsync pools are unaffected by
    // cudaDeviceEnablePeerAccess. We need pool-specific enablement. See
    // https://developer.nvidia.com/blog/using-cuda-stream-ordered-memory-allocator-part-2/
    c10::cuda::CUDAGuard device_guard(dev);
    cudaMemPool_t mempool = nullptr;
    C10_CUDA_CHECK(cudaDeviceGetDefaultMemPool(&mempool, dev_to_access));
    cudaMemAccessDesc desc = {};
    desc.location.type = cudaMemLocationTypeDevice;
    // NOLINTNEXTLINE(bugprone-signed-char-misuse)
    desc.location.id = dev;
    desc.flags = cudaMemAccessFlagsProtReadWrite;
    C10_CUDA_CHECK(cudaMemPoolSetAccess(mempool, &desc, 1 /* numDescs */));
  }
```
- **EN**: This chunk defines `device_guard`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `device_guard`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 906-929
```cpp
  cudaError_t memcpyAsync(
      void* dst,
      int dstDevice,
      const void* src,
      int srcDevice,
      size_t count,
      cudaStream_t stream,
      bool p2p_enabled) override {
    if (p2p_enabled || dstDevice == srcDevice) {
      return cudaMemcpyAsync(dst, src, count, cudaMemcpyDeviceToDevice, stream);
    } else {
      return cudaMemcpyPeerAsync(dst, dstDevice, src, srcDevice, count, stream);
    }
  }
  std::string name() override {
    // break up token to trick hipify
    return "c"
           "udaMallocAsync";
  }
  void copy_data(void* dest, const void* src, std::size_t count) const final {
    C10_CUDA_CHECK(
        cudaMemcpy(dest, src, count, cudaMemcpyKind::cudaMemcpyDeviceToDevice));
  }
};
```
- **EN**: This chunk defines `copy_data`, which duplicates state while preserving the ownership and metadata contracts. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `copy_data`，其作用是在保持所有权与元数据契约的前提下复制状态。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 931-941
```cpp
static CudaMallocAsyncAllocator device_allocator;

void local_raw_delete(void* ptr) {
  freeAsync(ptr);
}
// NOLINTNEXTLINE(misc-use-internal-linkage)
CUDAAllocator* allocator() {
  return &device_allocator;
}

} // namespace c10::cuda::CUDACachingAllocator::CudaMallocAsync
```
- **EN**: This chunk defines `freeAsync`, which manages allocation, reuse, or release decisions for runtime memory. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `freeAsync`，其作用是管理运行时内存的分配、复用或释放决策。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 末尾会返回累积结果，或把更新后的状态交还给调用方。


## Key Concepts / 关键概念
- **CUDA runtime support**
  - EN: Implements CUDA-facing c10 runtime support, including allocators, streams, guards, and device helpers.
  - CN: 实现面向 CUDA 的 c10 运行时支持，包括分配器、流、守卫以及设备辅助逻辑。
- **namespace**
  - EN: `namespace` is one of the dominant symbols declared or implemented in this file.
  - CN: `namespace` 是本文件声明或实现的关键符号之一。
- **UsageStream**
  - EN: `UsageStream` is one of the dominant symbols declared or implemented in this file.
  - CN: `UsageStream` 是本文件声明或实现的关键符号之一。
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
- **Internal includes / 内部依赖**: `c10/cuda/CUDACachingAllocator.h`、`c10/cuda/CUDAException.h`、`c10/cuda/CUDAFunctions.h`、`c10/cuda/CUDAGuard.h`、`c10/util/UniqueVoidPtr.h`、`c10/util/flat_hash_map.h`
- **Third-party includes / 第三方依赖**: 无
- **Standard includes / 标准库依赖**: `unordered_set`、`vector`
- **System includes / 系统依赖**: 无
- **Namespaces / 命名空间**: `c10::cuda::CUDACachingAllocator::CudaMallocAsync`
- **Representative symbols / 代表性符号**: `namespace`、`UsageStream`、`UsageStreamHash`、`PtrUsage`、`PtrInfo`、`static_cast<size_t>`、`lazy_init_device`、`g`、`getStreamFromPool`、`sync_raw`
