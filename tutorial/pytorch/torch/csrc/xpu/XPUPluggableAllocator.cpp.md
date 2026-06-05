# XPUPluggableAllocator.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `torch/csrc/xpu/XPUPluggableAllocator.cpp`
- **Repository**: `pytorch/pytorch`
- **Purpose**: This file implements logic for `XPUPluggableAllocator.cpp` inside the XPU backend bindings, memory helpers, streams, events, and graph utilities, with emphasis on allocator integration, xpu backend integration. / 该文件在XPU 后端绑定、内存辅助逻辑、流、事件与图工具中针对 `XPUPluggableAllocator.cpp` 实现逻辑，重点涉及分配器集成、XPU 后端集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2: Header dependencies / 头文件依赖
```cpp
#include <torch/csrc/xpu/XPUPluggableAllocator.h>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the XPU backend bindings, memory helpers, streams, events, and graph utilities.
- **CN**: 引入该翻译单元所需的头文件，包括来自XPU 后端绑定、内存辅助逻辑、流、事件与图工具的接口。

### Lines 3-4: Namespace scope / 命名空间作用域
```cpp
namespace torch::xpu::XPUPluggableAllocator {

```
- **EN**: Opens or organizes namespace scopes so related symbols live under the expected PyTorch component hierarchy.
- **CN**: 打开或组织命名空间作用域，使相关符号位于预期的 PyTorch 组件层级中。

### Lines 5-6: Supporting statements / 辅助语句
```cpp
void custom_raw_deleter(void* ptr);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 7-8: Supporting statements / 辅助语句
```cpp
static c10::DeviceIndex device_count_ = 0;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 9-20: Supporting statements / 辅助语句
```cpp
void* XPUPluggableAllocator::malloc(
    size_t size,
    c10::DeviceIndex device,
    sycl::queue* queue) {
  void* r = alloc_fn_(size, device, queue);
  {
    const std::lock_guard<std::mutex> lock(allocator_mutex_);
    allocation_metadata_.emplace(r, _AllocationMetadata(size, device, queue));
  }
  return r;
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 21-27: Function `allocate` / 函数 `allocate`
```cpp
c10::DataPtr XPUPluggableAllocator::allocate(size_t size) {
  auto device = c10::xpu::current_device();
  sycl::queue& queue = c10::xpu::getCurrentXPUStream(device);
  void* r = this->malloc(size, device, &queue);
  return {r, r, raw_deleter(), c10::Device(c10::kXPU, device)};
}

```
- **EN**: Implements `allocate`, one of the operational units in this file for the XPU backend bindings, memory helpers, streams, events, and graph utilities.
- **CN**: 实现 `allocate`，它是该文件中服务于XPU 后端绑定、内存辅助逻辑、流、事件与图工具的一个运行单元。

### Lines 28-33: Function `raw_alloc` / 函数 `raw_alloc`
```cpp
void* XPUPluggableAllocator::raw_alloc(size_t nbytes) {
  auto device = c10::xpu::current_device();
  sycl::queue& queue = c10::xpu::getCurrentXPUStream(device);
  return malloc(nbytes, device, &queue);
}

```
- **EN**: Implements `raw_alloc`, one of the operational units in this file for the XPU backend bindings, memory helpers, streams, events, and graph utilities.
- **CN**: 实现 `raw_alloc`，它是该文件中服务于XPU 后端绑定、内存辅助逻辑、流、事件与图工具的一个运行单元。

### Lines 34-37: Function `raw_deleter` / 函数 `raw_deleter`
```cpp
c10::DeleterFnPtr XPUPluggableAllocator::raw_deleter() const {
  return &custom_raw_deleter;
}

```
- **EN**: Implements `raw_deleter`, one of the operational units in this file for the XPU backend bindings, memory helpers, streams, events, and graph utilities.
- **CN**: 实现 `raw_deleter`，它是该文件中服务于XPU 后端绑定、内存辅助逻辑、流、事件与图工具的一个运行单元。

### Lines 38-55: Function `raw_delete` / 函数 `raw_delete`
```cpp
void XPUPluggableAllocator::raw_delete(void* ptr) {
  sycl::queue* queue = nullptr;
  c10::DeviceIndex device_idx = -1;
  size_t size = 0;
  {
    const std::lock_guard<std::mutex> lock(allocator_mutex_);
    TORCH_CHECK(
        allocation_metadata_.count(ptr),
        "Trying to free a pointer not allocated here");
    _AllocationMetadata& metadata = allocation_metadata_[ptr];
    size = metadata.size;
    device_idx = metadata.device_idx;
    queue = metadata.queue;
    allocation_metadata_.erase(ptr);
  }
  free_fn_(ptr, size, device_idx, queue);
}

```
- **EN**: Implements `raw_delete`, one of the operational units in this file for the XPU backend bindings, memory helpers, streams, events, and graph utilities.
- **CN**: 实现 `raw_delete`，它是该文件中服务于XPU 后端绑定、内存辅助逻辑、流、事件与图工具的一个运行单元。

### Lines 56-63: Function `init` / 函数 `init`
```cpp
void XPUPluggableAllocator::init(c10::DeviceIndex device_count) {
  if (init_fn_) {
    init_fn_(device_count);
  }
  device_count_ = device_count;
  initialized_ = true;
}

```
- **EN**: Implements `init`, one of the operational units in this file for the XPU backend bindings, memory helpers, streams, events, and graph utilities.
- **CN**: 实现 `init`，它是该文件中服务于XPU 后端绑定、内存辅助逻辑、流、事件与图工具的一个运行单元。

### Lines 64-67: Function `initialized` / 函数 `initialized`
```cpp
bool XPUPluggableAllocator::initialized() {
  return initialized_;
}

```
- **EN**: Implements `initialized`, one of the operational units in this file for the XPU backend bindings, memory helpers, streams, events, and graph utilities.
- **CN**: 实现 `initialized`，它是该文件中服务于XPU 后端绑定、内存辅助逻辑、流、事件与图工具的一个运行单元。

### Lines 68-74: Supporting statements / 辅助语句
```cpp
void XPUPluggableAllocator::copy_data(
    void* dest,
    const void* src,
    std::size_t count) const {
  c10::xpu::getCurrentXPUStream().queue().memcpy(dest, src, count);
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 75-83: Supporting statements / 辅助语句
```cpp
void XPUPluggableAllocator::recordStream(
    const c10::DataPtr& ptr,
    c10::Stream stream) {
  if (record_stream_fn_) {
    auto xpu_stream = c10::xpu::XPUStream(stream);
    record_stream_fn_(ptr.get(), &xpu_stream.queue());
  }
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 84-91: Supporting statements / 辅助语句
```cpp
void XPUPluggableAllocator::emptyCache(
    /*unused*/ c10::MempoolId_t mempool_id) {
  TORCH_CHECK(
      false,
      "XPUPluggableAllocator does not yet support emptyCache. "
      "If you need it, please file an issue describing your use case.");
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 92-99: Supporting statements / 辅助语句
```cpp
c10::CachingDeviceAllocator::DeviceStats XPUPluggableAllocator::getDeviceStats(
    c10::DeviceIndex device) {
  TORCH_CHECK(
      false,
      "XPUPluggableAllocator does not yet support getDeviceStats. "
      "If you need it, please file an issue describing your use case.");
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 100-106: Function `resetAccumulatedStats` / 函数 `resetAccumulatedStats`
```cpp
void XPUPluggableAllocator::resetAccumulatedStats(c10::DeviceIndex device) {
  TORCH_CHECK(
      false,
      "XPUPluggableAllocator does not yet support resetAccumulatedStats. "
      "If you need it, please file an issue describing your use case.");
}

```
- **EN**: Implements `resetAccumulatedStats`, one of the operational units in this file for the XPU backend bindings, memory helpers, streams, events, and graph utilities.
- **CN**: 实现 `resetAccumulatedStats`，它是该文件中服务于XPU 后端绑定、内存辅助逻辑、流、事件与图工具的一个运行单元。

### Lines 107-113: Function `resetPeakStats` / 函数 `resetPeakStats`
```cpp
void XPUPluggableAllocator::resetPeakStats(c10::DeviceIndex device) {
  TORCH_CHECK(
      false,
      "XPUPluggableAllocator does not yet support resetPeakStats. "
      "If you need it, please file an issue describing your use case.");
}

```
- **EN**: Implements `resetPeakStats`, one of the operational units in this file for the XPU backend bindings, memory helpers, streams, events, and graph utilities.
- **CN**: 实现 `resetPeakStats`，它是该文件中服务于XPU 后端绑定、内存辅助逻辑、流、事件与图工具的一个运行单元。

### Lines 114-116: Supporting statements / 辅助语句
```cpp
std::shared_ptr<c10::xpu::XPUCachingAllocator::XPUAllocator>
    current_custom_allocator;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 117-121: Supporting statements / 辅助语句
```cpp
std::shared_ptr<c10::xpu::XPUCachingAllocator::XPUAllocator>
getCurrentAllocator() {
  return current_custom_allocator;
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 122-131: Supporting statements / 辅助语句
```cpp
std::shared_ptr<c10::xpu::XPUCachingAllocator::XPUAllocator>
createCustomAllocator(
    std::function<void*(size_t, int, sycl::queue*)> alloc_fn,
    std::function<void(void*, size_t, int, sycl::queue*)> free_fn) {
  auto allocator = std::make_shared<XPUPluggableAllocator>(
      std::move(alloc_fn), std::move(free_fn));
  allocator->init(device_count_);
  return allocator;
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 132-142: Supporting statements / 辅助语句
```cpp
void changeCurrentAllocator(
    const std::shared_ptr<c10::xpu::XPUCachingAllocator::XPUAllocator>&
        allocator) {
  TORCH_CHECK(
      !c10::xpu::XPUCachingAllocator::get()->initialized(),
      "Can't swap an already initialized allocator");
  c10::xpu::XPUCachingAllocator::allocator.store(allocator.get());
  c10::SetAllocator(c10::kXPU, allocator.get());
  current_custom_allocator = allocator;
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 143-146: Function `custom_raw_deleter` / 函数 `custom_raw_deleter`
```cpp
void custom_raw_deleter(void* ptr) {
  current_custom_allocator->raw_delete(ptr);
}

```
- **EN**: Implements `custom_raw_deleter`, one of the operational units in this file for the XPU backend bindings, memory helpers, streams, events, and graph utilities.
- **CN**: 实现 `custom_raw_deleter`，它是该文件中服务于XPU 后端绑定、内存辅助逻辑、流、事件与图工具的一个运行单元。

### Lines 147-147: Supporting statements / 辅助语句
```cpp
} // namespace torch::xpu::XPUPluggableAllocator
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

## Key Concepts / 关键概念
- XPU backend integration / XPU 后端集成
- Allocator integration / 分配器集成

## Dependencies / 依赖关系
### Internal / 内部
- `torch/csrc/xpu/XPUPluggableAllocator.h`
### External / 外部
- None / 无
