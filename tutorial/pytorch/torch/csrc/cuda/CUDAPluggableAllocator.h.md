# CUDAPluggableAllocator.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `torch/csrc/cuda/CUDAPluggableAllocator.h`
- **Repository**: `pytorch/pytorch`
- **Purpose**: This file declares interfaces for `CUDAPluggableAllocator.h` inside the CUDA backend bindings, memory helpers, streams, events, and communication utilities, with emphasis on allocator integration, cuda backend integration. / 该文件在CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具中针对 `CUDAPluggableAllocator.h` 声明接口，重点涉及分配器集成、CUDA 后端集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2: Preprocessor configuration / 预处理配置
```cpp
#pragma once

```
- **EN**: Applies compile-time gating, macros, or platform-specific configuration so the file builds correctly under different feature sets.
- **CN**: 应用编译期开关、宏或平台特定配置，使该文件能在不同功能组合下正确构建。

### Lines 3-7: Header dependencies / 头文件依赖
```cpp
#include <c10/core/Allocator.h>
#include <c10/cuda/CUDAGraphsC10Utils.h>
#include <c10/cuda/CUDAMacros.h>
#include <c10/cuda/CUDAStream.h>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 引入该翻译单元所需的头文件，包括来自CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的接口。

### Lines 8-9: Header dependencies / 头文件依赖
```cpp
#include <c10/cuda/CUDACachingAllocator.h>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 引入该翻译单元所需的头文件，包括来自CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的接口。

### Lines 10-11: Header dependencies / 头文件依赖
```cpp
#include <mutex>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 引入该翻译单元所需的头文件，包括来自CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的接口。

### Lines 12-13: Namespace scope / 命名空间作用域
```cpp
namespace torch::cuda::CUDAPluggableAllocator {

```
- **EN**: Opens or organizes namespace scopes so related symbols live under the expected PyTorch component hierarchy.
- **CN**: 打开或组织命名空间作用域，使相关符号位于预期的 PyTorch 组件层级中。

### Lines 14-25: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API std::shared_ptr<
    c10::cuda::CUDACachingAllocator::CUDAAllocator>
getCurrentAllocator();
TORCH_CUDA_CPP_API std::shared_ptr<
    c10::cuda::CUDACachingAllocator::CUDAAllocator>
createCustomAllocator(
    std::function<void*(size_t, int, cudaStream_t)> alloc_fn,
    std::function<void(void*, size_t, int, cudaStream_t)> free_fn);
TORCH_CUDA_CPP_API void changeCurrentAllocator(
    const std::shared_ptr<c10::cuda::CUDACachingAllocator::CUDAAllocator>&
        allocator);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 26-36: Type declaration / 类型声明
```cpp
struct _AllocationMetadata {
  _AllocationMetadata();
  _AllocationMetadata(
      size_t size,
      c10::DeviceIndex device_idx,
      cudaStream_t stream);
  size_t size;
  c10::DeviceIndex device_idx;
  cudaStream_t stream{};
};

```
- **EN**: Declares a core helper type, wrapper, or policy object that structures the rest of the implementation.
- **CN**: 声明一个核心辅助类型、封装对象或策略对象，用于组织后续实现。

### Lines 37-42: Type declaration / 类型声明
```cpp
struct TORCH_CUDA_CPP_API CUDAPluggableAllocator
    : public c10::cuda::CUDACachingAllocator::CUDAAllocator {
  CUDAPluggableAllocator(
      std::function<void*(size_t, int, cudaStream_t)> alloc_fn,
      std::function<void(void*, size_t, int, cudaStream_t)> free_fn);

```
- **EN**: Declares a core helper type, wrapper, or policy object that structures the rest of the implementation.
- **CN**: 声明一个核心辅助类型、封装对象或策略对象，用于组织后续实现。

### Lines 43-49: Supporting statements / 辅助语句
```cpp
  CUDAPluggableAllocator(CUDAPluggableAllocator& other);
  CUDAPluggableAllocator(CUDAPluggableAllocator&& other) = delete;
  CUDAPluggableAllocator& operator=(const CUDAPluggableAllocator& other) =
      delete;
  CUDAPluggableAllocator& operator=(CUDAPluggableAllocator&& other) = delete;
  ~CUDAPluggableAllocator() override = default;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 50-51: Supporting statements / 辅助语句
```cpp
  void set_init_fn(std::function<void(int)> init_fn);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 52-53: Supporting statements / 辅助语句
```cpp
  void set_reset_fn(std::function<void()> reset_fn);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 54-56: Supporting statements / 辅助语句
```cpp
  void set_memory_fraction_fn(
      std::function<void(double, int)> memory_fraction_fn);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 57-58: Supporting statements / 辅助语句
```cpp
  void set_base_alloc_fn(std::function<void*(void*, size_t*)> base_alloc_fn);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 59-61: Supporting statements / 辅助语句
```cpp
  void set_record_stream_fn(
      std::function<void(void* ptr, cudaStream_t stream)> record_stream_fn);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 62-66: Supporting statements / 辅助语句
```cpp
  void set_begin_allocate_to_pool(
      std::function<
          void(int, c10::cuda::MempoolId_t, std::function<bool(cudaStream_t)>)>
          capture_begin_fn);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 67-69: Supporting statements / 辅助语句
```cpp
  void set_end_allocate_to_pool_fn(
      std::function<void(int, c10::cuda::MempoolId_t)> capture_about_to_end_fn);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 70-72: Supporting statements / 辅助语句
```cpp
  void set_release_pool(
      std::function<void(int, c10::cuda::MempoolId_t)> capture_destroy_fn);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 73-74: Supporting statements / 辅助语句
```cpp
  void* malloc(size_t size, c10::DeviceIndex device, cudaStream_t stream);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 75-77: Supporting statements / 辅助语句
```cpp
  c10::DataPtr allocate(size_t size) override;
  c10::DeleterFnPtr raw_deleter() const override;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 78-94: Supporting statements / 辅助语句
```cpp
  void* raw_alloc(size_t nbytes) override;
  void* raw_alloc_with_stream(size_t nbytes, cudaStream_t stream) override;
  void raw_delete(void* ptr) override;
  void init(int device_count) override;
  bool initialized() override;
  double getMemoryFraction(c10::DeviceIndex device) override;
  void setMemoryFraction(double fraction, c10::DeviceIndex device) override;
  std::vector<c10::cuda::CUDACachingAllocator::StreamSegmentSize>
  getExpandableSegmentSizes(c10::DeviceIndex device) override;
  void emptyCache(c10::cuda::MempoolId_t mempool_id = {0, 0}) override;
  void enable(bool /*value*/) override {}
  bool isEnabled() const override {
    return true;
  }
  void cacheInfo(c10::DeviceIndex device, size_t* largestBlock) override;
  void* getBaseAllocation(void* ptr, size_t* size) override;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 95-97: Function `recordStream` / 函数 `recordStream`
```cpp
  void recordStream(const c10::DataPtr& /*ptr*/, c10::cuda::CUDAStream stream)
      override;

```
- **EN**: Implements `recordStream`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `recordStream`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 98-121: Supporting statements / 辅助语句
```cpp
  c10::CachingDeviceAllocator::DeviceStats getDeviceStats(
      c10::DeviceIndex device) override;
  void resetAccumulatedStats(c10::DeviceIndex device) override;
  void resetPeakStats(c10::DeviceIndex device) override;
  c10::cuda::CUDACachingAllocator::SnapshotInfo snapshot(
      c10::cuda::MempoolId_t mempool,
      bool include_traces = true) override;
  void beginAllocateToPool(
      c10::DeviceIndex device,
      c10::cuda::MempoolId_t mempool_id,
      std::function<bool(cudaStream_t)> /*filter*/) override;
  void endAllocateToPool(
      c10::DeviceIndex device,
      c10::cuda::MempoolId_t mempool_id) override;
  void releasePool(c10::DeviceIndex device, c10::cuda::MempoolId_t mempool_id)
      override;
  std::shared_ptr<void> getIpcDevPtr(std::string handle) override;
  c10::cuda::CUDACachingAllocator::ShareableHandle shareIpcHandle(
      void* /*ptr*/) override;
  void recordHistory(
      bool enabled,
      c10::cuda::CUDACachingAllocator::CreateContextFn context_recorder,
      size_t alloc_trace_max_entries,
      c10::cuda::CUDACachingAllocator::RecordContext when,
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 122-145: Supporting statements / 辅助语句
```cpp
      bool clearHistory,
      const std::vector<std::string>& skip_actions) override;
  void attachOutOfMemoryObserver(
      c10::cuda::CUDACachingAllocator::OutOfMemoryObserver observer) override;
  void attachOomRejectionObserver(
      c10::cuda::CUDACachingAllocator::OomRejectionObserver observer) override;
  void attachAllocatorTraceTracker(
      c10::cuda::CUDACachingAllocator::AllocatorTraceTracker tracker) override;
  std::shared_ptr<c10::cuda::CUDACachingAllocator::AllocatorState>
  getCheckpointState(c10::DeviceIndex device, at::cuda::MempoolId_t id)
      override;
  c10::cuda::CUDACachingAllocator::CheckpointDelta setCheckpointPoolState(
      c10::DeviceIndex device,
      std::shared_ptr<c10::cuda::CUDACachingAllocator::AllocatorState> pps)
      override;
  void enablePeerAccess(c10::DeviceIndex dev, c10::DeviceIndex dev_to_access)
      override;
  cudaError_t memcpyAsync(
      void* dst,
      int dstDevice,
      const void* src,
      int srcDevice,
      size_t count,
      cudaStream_t stream,
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 146-149: Supporting statements / 辅助语句
```cpp
      bool p2p_enabled) override;
  std::string name() override;
  void copy_data(void* dest, const void* src, std::size_t count) const final;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 150-166: Supporting statements / 辅助语句
```cpp
 protected:
  std::function<void*(size_t, int, cudaStream_t)> alloc_fn_;
  std::function<void(void*, size_t, int, cudaStream_t)> free_fn_;
  std::function<void(int)> init_fn_;
  std::function<void()> reset_fn_;
  std::function<void(double, int)> memory_fraction_fn_;
  std::function<void*(void*, size_t*)> base_alloc_fn_;
  std::function<void(void* ptr, cudaStream_t stream)> record_stream_fn_;
  std::function<
      void(int, c10::cuda::MempoolId_t, std::function<bool(cudaStream_t)>)>
      begin_allocate_to_pool_fn_;
  std::function<void(int, c10::cuda::MempoolId_t)> end_allocate_to_pool_fn_;
  std::function<void(int, c10::cuda::MempoolId_t)> relase_pool_fn_;
  std::mutex allocator_mutex_;
  // We do the bookkeeping here in order to simplify custom allocators
  std::unordered_map<void*, _AllocationMetadata> allocation_metadata_;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 167-169: Supporting statements / 辅助语句
```cpp
  bool initialized_ = false;
};
} // namespace torch::cuda::CUDAPluggableAllocator
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

## Key Concepts / 关键概念
- CUDA backend integration / CUDA 后端集成
- Allocator integration / 分配器集成

## Dependencies / 依赖关系
### Internal / 内部
- `c10/core/Allocator.h`
- `c10/cuda/CUDAGraphsC10Utils.h`
- `c10/cuda/CUDAMacros.h`
- `c10/cuda/CUDAStream.h`
- `c10/cuda/CUDACachingAllocator.h`
### External / 外部
- `mutex`
