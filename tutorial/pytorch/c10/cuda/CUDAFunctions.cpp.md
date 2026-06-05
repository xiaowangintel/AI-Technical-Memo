# CUDAFunctions.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `c10/cuda/CUDAFunctions.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-specific runtime helpers, allocators, stream utilities, and error handling.
- **Purpose (CN)**: 实现 CUDA 专用的运行时辅助逻辑、分配器、流工具与错误处理。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15
```cpp
#include <c10/cuda/CUDAFunctions.h>
#include <c10/macros/Macros.h>
#include <c10/util/WaitCounter.h>

#include <limits>

namespace c10::cuda {

namespace {
// returns -1 on failure
int32_t driver_version() {
  int driver_version = -1;
  C10_CUDA_IGNORE_ERROR(cudaDriverGetVersion(&driver_version));
  return driver_version;
}
```
- **EN**: This block assembles the compilation dependencies, pulling in local PyTorch/c10 headers such as c10/cuda/CUDAFunctions.h, c10/macros/Macros.h, c10/util/WaitCounter.h; standard-library headers such as limits. The namespace declarations place the code inside c10::cuda, matching the surrounding subsystem. This chunk defines `driver_version`, which implements a focused piece of backend/runtime support logic. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段组织编译依赖，引入了本地 PyTorch/c10 头文件，如 c10/cuda/CUDAFunctions.h、c10/macros/Macros.h、c10/util/WaitCounter.h；标准库头文件，如 limits。 命名空间声明把代码放入 c10::cuda 中，与周边子系统保持一致。 这一段定义了 `driver_version`，其作用是实现一段聚焦的后端/运行时支持逻辑。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 17-34
```cpp
int device_count_impl(bool fail_if_no_driver) {
  int count = 0;
  auto err = C10_CUDA_ERROR_HANDLED(c10::cuda::GetDeviceCount(&count));
  if (err == cudaSuccess) {
    return count;
  }
  // Clear out the error state, so we don't spuriously trigger someone else.
  // (This shouldn't really matter, since we won't be running very much CUDA
  // code in this regime.)
  [[maybe_unused]] cudaError_t last_err = cudaGetLastError();
  switch (err) {
    case cudaErrorNoDevice:
      // Zero devices is ok here
      count = 0;
      break;
    case cudaErrorInsufficientDriver: {
      auto version = driver_version();
      if (version <= 0) {
```
- **EN**: This chunk defines `driver_version`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Switch-based dispatch selects specialized behavior for enum variants, backend kinds, or opcode-like cases. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `driver_version`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 基于 switch 的分发会为枚举分支、后端类型或类似 opcode 的场景选择专门行为。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 35-52
```cpp
        if (!fail_if_no_driver) {
          // No CUDA driver means no devices
          count = 0;
          break;
        }
        TORCH_CHECK(
            false,
            "Found no NVIDIA driver on your system. Please check that you "
            "have an NVIDIA GPU and installed a driver from "
            "http://www.nvidia.com/Download/index.aspx");
      } else {
        TORCH_CHECK(
            false,
            "The NVIDIA driver on your system is too old (found version ",
            version,
            "). Please update your GPU driver by downloading and installing "
            "a new version from the URL: "
            "http://www.nvidia.com/Download/index.aspx Alternatively, go to: "
```
- **EN**: This chunk continues `driver_version` and expands its control flow, data movement, or edge-case handling. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段延续了 `driver_version`，进一步展开其控制流、数据流转或边界处理逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 53-70
```cpp
            "https://pytorch.org to install a PyTorch version that has been "
            "compiled with your version of the CUDA driver.");
      }
    }
    case cudaErrorInitializationError:
      TORCH_CHECK(
          false,
          "CUDA driver initialization failed, you might not "
          "have a CUDA gpu.");
    case cudaErrorUnknown:
      TORCH_CHECK(
          false,
          "CUDA unknown error - this may be due to an "
          "incorrectly set up environment, e.g. changing env "
          "variable CUDA_VISIBLE_DEVICES after program start. "
          "Setting the available devices to be zero.");
#if C10_ASAN_ENABLED
    case cudaErrorMemoryAllocation:
```
- **EN**: This chunk continues `driver_version` and expands its control flow, data movement, or edge-case handling. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段延续了 `driver_version`，进一步展开其控制流、数据流转或边界处理逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 71-88
```cpp
      // In ASAN mode, we know that a cudaErrorMemoryAllocation error will
      // pop up if compiled with NVCC (clang-cuda is fine)
      TORCH_CHECK(
          false,
          "Got 'out of memory' error while trying to initialize CUDA. "
          "CUDA with nvcc does not work well with ASAN and it's probably "
          "the reason. We will simply shut down CUDA support. If you "
          "would like to use GPUs, turn off ASAN.");
      break;
#endif // C10_ASAN_ENABLED
#if defined(_WIN32) && CUDA_VERSION >= 13000
    // Workaround for CUDA-13.0 error handling on Windows, see
    // https://github.com/pytorch/pytorch/issues/162333#issuecomment-3267929585
    case cudaErrorNotSupported:
      if (!fail_if_no_driver) {
        TORCH_WARN(
            "cudaGetDeviceCount() returned cudaErrorNotSupported, "
            "likely using older driver or on CPU machine");
```
- **EN**: It introduces or extends older, which define the main data structures or interfaces for this portion of the file. This chunk defines `defined`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 它引入或扩展了 older，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `defined`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 89-105
```cpp
        count = 0;
        break;
      }
#endif
    default:
      TORCH_CHECK(
          false,
          "Unexpected error from cudaGetDeviceCount(). Did you run "
          "some cuda functions before calling NumCudaDevices() "
          "that might have already set an error? Error ",
          err,
          ": ",
          cudaGetErrorString(err));
  }
  return count;
}
} // namespace
```
- **EN**: This chunk continues `defined` and expands its control flow, data movement, or edge-case handling. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段延续了 `defined`，进一步展开其控制流、数据流转或边界处理逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 107-124
```cpp
DeviceIndex device_count() noexcept {
  // initialize number of devices only once
  static int count = []() {
    try {
      auto result = device_count_impl(/*fail_if_no_driver=*/false);
      TORCH_INTERNAL_ASSERT(
          result <= std::numeric_limits<DeviceIndex>::max(),
          "Too many CUDA devices, DeviceIndex overflowed");
      return result;
    } catch (const c10::Error& ex) {
      // We don't want to fail, but still log the warning
      // msg() returns the message without the stack trace
      TORCH_WARN("CUDA initialization: ", ex.msg());
      return 0;
    }
  }();
  return static_cast<DeviceIndex>(count);
}
```
- **EN**: This chunk defines `static_cast<DeviceIndex>`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `static_cast<DeviceIndex>`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 126-141
```cpp
DeviceIndex device_count_ensure_non_zero() {
  // Call the implementation every time to throw the exception
  int count = device_count_impl(/*fail_if_no_driver=*/true);
  // Zero gpus doesn't produce a warning in `device_count` but we fail here
  TORCH_CHECK(count, "No CUDA GPUs are available");
  TORCH_INTERNAL_ASSERT(
      count <= std::numeric_limits<DeviceIndex>::max(),
      "Too many CUDA devices, DeviceIndex overflowed");
  return static_cast<DeviceIndex>(count);
}

DeviceIndex current_device() {
  DeviceIndex cur_device = -1;
  C10_CUDA_CHECK(c10::cuda::GetDevice(&cur_device));
  return cur_device;
}
```
- **EN**: This chunk defines `current_device`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `current_device`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 143-154
```cpp
void set_device(DeviceIndex device, const bool force) {
  C10_CUDA_CHECK(c10::cuda::SetDevice(device, force));
}

void device_synchronize() {
  const c10::impl::PyInterpreter* interp = c10::impl::GPUTrace::get_trace();
  if (C10_UNLIKELY(interp)) {
    (*interp)->trace_gpu_device_synchronization(c10::kCUDA);
  }
  STATIC_SCOPED_WAIT_COUNTER(pytorch.wait_counter.cuda_device_synchronize);
  C10_CUDA_CHECK(cudaDeviceSynchronize());
}
```
- **EN**: This chunk defines `trace_gpu_device_synchronization`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `trace_gpu_device_synchronization`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 156-173
```cpp
// this function has to be called from callers performing cuda synchronizing
// operations, to raise proper error or warning
void warn_or_error_on_sync() {
  if (warning_state().get_sync_debug_mode() == SyncDebugMode::L_ERROR) {
    TORCH_CHECK(false, "called a synchronizing CUDA operation");
  } else if (warning_state().get_sync_debug_mode() == SyncDebugMode::L_WARN) {
    TORCH_WARN("called a synchronizing CUDA operation");
  }
}

std::optional<DeviceIndex> getDeviceIndexWithPrimaryContext() {
  // check current device first
  auto current_device_index = current_device();
  if (current_device_index >= 0) {
    if (hasPrimaryContext(current_device_index)) {
      return current_device_index;
    }
  }
```
- **EN**: This chunk defines `current_device`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `current_device`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 174-188
```cpp
  for (const auto device_index : c10::irange(at::cuda::device_count())) {
    if (device_index == current_device_index)
      continue;
    if (hasPrimaryContext(device_index)) {
      return device_index;
    }
  }
  return std::nullopt;
}

namespace _internal {
static bool dummyHasPrimaryContext([[maybe_unused]] DeviceIndex device_index) {
  TORCH_CHECK(false, "Should never been called");
}
static bool (*hasPrimaryContext)(DeviceIndex) = dummyHasPrimaryContext;
```
- **EN**: The namespace declarations place the code inside _internal, matching the surrounding subsystem. This chunk defines `dummyHasPrimaryContext`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 命名空间声明把代码放入 _internal 中，与周边子系统保持一致。 这一段定义了 `dummyHasPrimaryContext`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 190-204
```cpp
// Private api to be called from CUDAHooks.cpp
// NOLINTNEXTLINE(misc-use-internal-linkage)
C10_CUDA_API void setHasPrimaryContext(bool (*func)(DeviceIndex)) {
  hasPrimaryContext = func ? func : dummyHasPrimaryContext;
}
} // namespace _internal

bool hasPrimaryContext(DeviceIndex device_index) {
  return _internal::hasPrimaryContext(device_index);
}

// Wrappers for raw CUDA device management functions
cudaError_t GetDeviceCount(int* dev_count) {
  return cudaGetDeviceCount(dev_count);
}
```
- **EN**: This chunk defines `cudaGetDeviceCount`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `cudaGetDeviceCount`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 206-223
```cpp
// This is a codepath for CUDA 12 that comes with a critical change in behavior
// of `cudaSetDevice`. Unlike to previous CUDA versions that allocate context
// lazily CUDA 12.x eagerly allocates primary context the moment `cudaSetDevice`
// is called. This can lead to dramatic consequences and pollute the device
// memory in distributed runs. To avoid unnecessary context creation a new
// function called `MaybeSetDevice` was introduced. This function is to be
// called in device guard destructor and at the exit of torch.cuda.device
// context manager. The behavior of `MaybeSetDevice` is quite simple, it calls
// to `cudaSetDevice` if context already exist or if context was not allocated
// on targeted device it simply saves the device index. This way we can keep
// PyTorch backward compatible for applications like this:
//
// ```
// import torch
// x = torch.empty(1, device=“cuda:1”) # no CUDA context on cuda:0 after this
// call y = torch.empty(1, device=“cuda”) # CUDA context is created on cuda:0
// ```
#if CUDA_VERSION >= 12000
```
- **EN**: This chunk continues `cudaGetDeviceCount` and expands its control flow, data movement, or edge-case handling. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 这一段延续了 `cudaGetDeviceCount`，进一步展开其控制流、数据流转或边界处理逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 224-240
```cpp
thread_local static DeviceIndex targetDeviceIndex = -1;

cudaError_t GetDevice(DeviceIndex* device) {
  if (targetDeviceIndex >= 0) {
    *device = targetDeviceIndex;
    return cudaSuccess;
  }
  int tmp_device = -1;
  auto err = cudaGetDevice(&tmp_device);
  if (err == cudaSuccess) {
    TORCH_INTERNAL_ASSERT(
        tmp_device >= 0 &&
            tmp_device <= std::numeric_limits<DeviceIndex>::max(),
        "cudaGetDevice returns invalid device ",
        tmp_device);
    *device = static_cast<DeviceIndex>(tmp_device);
  }
```
- **EN**: This chunk defines `static_cast<DeviceIndex>`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `static_cast<DeviceIndex>`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 241-257
```cpp
  return err;
}

cudaError_t SetDevice(DeviceIndex device, const bool force) {
  TORCH_CHECK(
      device >= 0, "device id must be non-negative!", static_cast<int>(device));
  targetDeviceIndex = -1;
  if (force) {
    return cudaSetDevice(device);
  }
  int cur_device = -1;
  C10_CUDA_CHECK(cudaGetDevice(&cur_device));
  if (device == cur_device) {
    return cudaSuccess;
  }
  return cudaSetDevice(device);
}
```
- **EN**: This chunk defines `cudaSetDevice`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `cudaSetDevice`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 259-276
```cpp
cudaError_t MaybeSetDevice(DeviceIndex device) {
  if (hasPrimaryContext(device)) {
    return c10::cuda::SetDevice(device);
  }
  targetDeviceIndex = device;
  return cudaSuccess;
}

// This function always initializes the CUDA context
// on to_device
DeviceIndex ExchangeDevice(DeviceIndex to_device) {
  auto cur_device = targetDeviceIndex;
  targetDeviceIndex = -1;
  if (cur_device < 0) {
    int tmp_device = -1;
    C10_CUDA_CHECK(cudaGetDevice(&tmp_device));
    cur_device = static_cast<DeviceIndex>(tmp_device);
    if (to_device == cur_device) {
```
- **EN**: This chunk defines `static_cast<DeviceIndex>`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `static_cast<DeviceIndex>`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 277-294
```cpp
      return cur_device;
    }
  }
  C10_CUDA_CHECK(cudaSetDevice(to_device));
  return cur_device;
}

// This function does not initialize the CUDA context
// on to_device if it does not already exist
DeviceIndex MaybeExchangeDevice(DeviceIndex to_device) {
  int tmp_cur_device = -1;
  C10_CUDA_CHECK(cudaGetDevice(&tmp_cur_device));
  TORCH_INTERNAL_ASSERT(
      tmp_cur_device >= 0 &&
          tmp_cur_device <= std::numeric_limits<DeviceIndex>::max(),
      "cudaGetDevice returns invalid device ",
      tmp_cur_device);
  auto cur_device = static_cast<DeviceIndex>(tmp_cur_device);
```
- **EN**: This chunk defines `static_cast<DeviceIndex>`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `static_cast<DeviceIndex>`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 295-310
```cpp
  if (to_device == tmp_cur_device) {
    return cur_device;
  }
  if (hasPrimaryContext(to_device)) {
    C10_CUDA_CHECK(cudaSetDevice(to_device));
  } else {
    targetDeviceIndex = to_device;
  }
  return cur_device;
}

void SetTargetDevice() {
  if (targetDeviceIndex >= 0) {
    C10_CUDA_CHECK(c10::cuda::SetDevice(targetDeviceIndex));
  }
}
```
- **EN**: This chunk defines `SetTargetDevice`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `SetTargetDevice`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 311-324
```cpp
#else
cudaError_t GetDevice(DeviceIndex* device) {
  int tmp_device = -1;
  auto err = cudaGetDevice(&tmp_device);
  if (err == cudaSuccess) {
    TORCH_INTERNAL_ASSERT(
        tmp_device >= 0 &&
            tmp_device <= std::numeric_limits<DeviceIndex>::max(),
        "cudaGetDevice returns invalid device ",
        tmp_device);
    *device = static_cast<DeviceIndex>(tmp_device);
  }
  return err;
}
```
- **EN**: This chunk defines `static_cast<DeviceIndex>`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `static_cast<DeviceIndex>`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 326-342
```cpp
cudaError_t SetDevice(DeviceIndex device, const bool force) {
  TORCH_CHECK(
      device >= 0, "device id must be non-negative!", static_cast<int>(device));
  if (force) {
    return cudaSetDevice(device);
  }
  int cur_device = -1;
  C10_CUDA_CHECK(cudaGetDevice(&cur_device));
  if (device == cur_device) {
    return cudaSuccess;
  }
  return cudaSetDevice(device);
}

cudaError_t MaybeSetDevice(DeviceIndex device) {
  return c10::cuda::SetDevice(device);
}
```
- **EN**: This chunk defines `MaybeSetDevice`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `MaybeSetDevice`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 344-360
```cpp
DeviceIndex ExchangeDevice(DeviceIndex to_device) {
  DeviceIndex cur_device = -1;
  C10_CUDA_CHECK(c10::cuda::GetDevice(&cur_device));
  if (to_device == cur_device) {
    return cur_device;
  }
  C10_CUDA_CHECK(cudaSetDevice(to_device));
  return cur_device;
}

DeviceIndex MaybeExchangeDevice(DeviceIndex to_device) {
  return c10::cuda::ExchangeDevice(to_device);
}

void SetTargetDevice() {
  // no-op on CUDA version < 12.x
}
```
- **EN**: This chunk defines `SetTargetDevice`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `SetTargetDevice`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 361-363
```cpp
#endif

} // namespace c10::cuda
```
- **EN**: This chunk continues `SetTargetDevice` and expands its control flow, data movement, or edge-case handling. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules.
- **CN**: 这一段延续了 `SetTargetDevice`，进一步展开其控制流、数据流转或边界处理逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。


## Key Concepts / 关键概念
- **CUDA runtime support**
  - EN: Implements CUDA-facing c10 runtime support, including allocators, streams, guards, and device helpers.
  - CN: 实现面向 CUDA 的 c10 运行时支持，包括分配器、流、守卫以及设备辅助逻辑。
- **older**
  - EN: `older` is one of the dominant symbols declared or implemented in this file.
  - CN: `older` 是本文件声明或实现的关键符号之一。
- **driver_version**
  - EN: `driver_version` is one of the dominant symbols declared or implemented in this file.
  - CN: `driver_version` 是本文件声明或实现的关键符号之一。
- **Device abstraction**
  - EN: Represents backend/device identity so code can stay portable across runtimes.
  - CN: 表示后端/设备身份，使代码在不同运行时之间保持可移植。
- **Error handling**
  - EN: Surfaces invalid states through assertions, exceptions, and diagnostic messages.
  - CN: 通过断言、异常与诊断消息暴露非法状态。
- **CUDA integration**
  - EN: Connects c10 abstractions to CUDA allocators, streams, and device/runtime APIs.
  - CN: 把 c10 抽象连接到 CUDA 分配器、流以及设备/运行时 API。

## Dependencies / 依赖关系
- **Internal includes / 内部依赖**: `c10/cuda/CUDAFunctions.h`、`c10/macros/Macros.h`、`c10/util/WaitCounter.h`
- **Third-party includes / 第三方依赖**: 无
- **Standard includes / 标准库依赖**: `limits`
- **System includes / 系统依赖**: 无
- **Namespaces / 命名空间**: `c10::cuda`、`_internal`
- **Representative symbols / 代表性符号**: `older`、`driver_version`、`device_count_impl`、`cudaGetLastError`、`defined`、`device_count`、`msg`、`static_cast<DeviceIndex>`、`device_count_ensure_non_zero`、`current_device`
