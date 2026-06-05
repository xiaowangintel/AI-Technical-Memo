# PeerToPeerAccess.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `c10/cuda/PeerToPeerAccess.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-specific runtime helpers, allocators, stream utilities, and error handling.
- **Purpose (CN)**: 实现 CUDA 专用的运行时辅助逻辑、分配器、流工具与错误处理。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17
```cpp
#include <c10/cuda/PeerToPeerAccess.h>

#include <c10/cuda/CUDACachingAllocator.h>
#include <c10/cuda/CUDAException.h>
#include <c10/cuda/CUDAGuard.h>
#if !defined(USE_ROCM) && defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
#include <c10/cuda/driver_api.h>
#endif
#include <c10/util/Exception.h>
#include <c10/util/Logging.h>
#include <c10/util/irange.h>

#include <iomanip>
#include <sstream>
#include <vector>

namespace c10::cuda {
```
- **EN**: This block assembles the compilation dependencies, pulling in local PyTorch/c10 headers such as c10/cuda/PeerToPeerAccess.h, c10/cuda/CUDACachingAllocator.h, c10/cuda/CUDAException.h, and 5 more; standard-library headers such as iomanip, sstream, vector. The namespace declarations place the code inside c10::cuda, matching the surrounding subsystem. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段组织编译依赖，引入了本地 PyTorch/c10 头文件，如 c10/cuda/PeerToPeerAccess.h、c10/cuda/CUDACachingAllocator.h、c10/cuda/CUDAException.h 等共 8 项；标准库头文件，如 iomanip、sstream、vector。 命名空间声明把代码放入 c10::cuda 中，与周边子系统保持一致。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 19-34
```cpp
static std::vector<int8_t> p2pAccessEnabled_;
static std::vector<int8_t> fabricAccessEnabled_;
static std::vector<int> fabricCliqueId_;
static int64_t num_devices_ = -1;

namespace detail {

void init_p2p_access_cache(int64_t num_devices) {
  // p2pAccessEnabled records if p2p copies are allowed between pairs of
  // devices. Values include "1" (copy allowed), "0" (copy not allowed), and
  // "-1" (unknown).
  // Currently the max number of gpus in P2P group is 8, so if there are more
  // we enable P2P in groups of 8
  p2pAccessEnabled_.clear();
  p2pAccessEnabled_.resize(num_devices * num_devices, -1);
  num_devices_ = num_devices;
```
- **EN**: The namespace declarations place the code inside detail, matching the surrounding subsystem. This chunk defines `resize`, which implements a focused piece of backend/runtime support logic. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 命名空间声明把代码放入 detail 中，与周边子系统保持一致。 这一段定义了 `resize`，其作用是实现一段聚焦的后端/运行时支持逻辑。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 36-53
```cpp
  for (const auto i : c10::irange(num_devices)) {
    p2pAccessEnabled_[i * num_devices + i] = 1;
  }
  fabricAccessEnabled_.clear();
  fabricAccessEnabled_.resize(num_devices, -1);
  fabricCliqueId_.clear();
  fabricCliqueId_.resize(num_devices, kCliqueIdNotQueried);
}

} // namespace detail

bool get_p2p_access(c10::DeviceIndex dev, c10::DeviceIndex dev_to_access) {
  TORCH_CHECK(
      num_devices_ >= 0,
      "p2p access cache not initialized. "
      "Ensure c10::cuda::detail::init_p2p_access_cache() is called first.");
  TORCH_CHECK(
      dev >= 0 && dev < num_devices_,
```
- **EN**: This chunk defines `get_p2p_access`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `get_p2p_access`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 54-65
```cpp
      static_cast<int>(dev),
      " is not a valid device");
  TORCH_CHECK(
      dev_to_access >= 0 && dev_to_access < num_devices_,
      static_cast<int>(dev_to_access),
      " is not a valid device");

  auto& cache = p2pAccessEnabled_[dev * num_devices_ + dev_to_access];

  if (cache != -1) {
    return cache;
  }
```
- **EN**: This chunk defines `static_cast<int>`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `static_cast<int>`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 67-79
```cpp
  int result = 0;
  C10_CUDA_CHECK(cudaDeviceCanAccessPeer(&result, dev, dev_to_access));
  cache = result ? 1 : 0;
  if (cache) {
    CUDACachingAllocator::enablePeerAccess(dev, dev_to_access);
  }

  return cache;
}

namespace {
#if !defined(USE_ROCM) && defined(CUDA_VERSION) && CUDA_VERSION >= 12040 && \
    defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
```
- **EN**: This chunk defines `enablePeerAccess`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `enablePeerAccess`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 81-98
```cpp
nvmlDevice_t get_nvml_device(c10::DeviceIndex dev) {
  static bool nvml_init [[maybe_unused]] = []() {
    TORCH_INTERNAL_ASSERT(NVML_SUCCESS == DriverAPI::get()->nvmlInit_v2_());
    return true;
  }();

  // Direct CUDA call instead of at::cuda::getDeviceProperties()
  cudaDeviceProp prop{};
  C10_CUDA_CHECK(cudaGetDeviceProperties(&prop, dev));

  char
      pci_id // NOLINT(cppcoreguidelines-avoid-c-arrays,modernize-avoid-c-arrays)
          [NVML_DEVICE_PCI_BUS_ID_BUFFER_SIZE];
  snprintf(
      pci_id,
      sizeof(pci_id),
      NVML_DEVICE_PCI_BUS_ID_FMT,
      prop.pciDomainID,
```
- **EN**: This chunk defines `get_nvml_device`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `get_nvml_device`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 99-116
```cpp
      prop.pciBusID,
      prop.pciDeviceID);

  nvmlDevice_t nvml_device = nullptr;
  TORCH_INTERNAL_ASSERT(
      NVML_SUCCESS ==
      DriverAPI::get()->nvmlDeviceGetHandleByPciBusId_v2_(
          pci_id, &nvml_device));
  return nvml_device;
}

bool isFabricSupported() {
  // 1. try allocating memory with FABRIC handle type
  CUmemGenericAllocationHandle handle = 0;
  CUmemAllocationProp prop = {};
  prop.type = CU_MEM_ALLOCATION_TYPE_PINNED;
  prop.requestedHandleTypes = CU_MEM_HANDLE_TYPE_FABRIC;
  prop.location.type = CU_MEM_LOCATION_TYPE_DEVICE;
```
- **EN**: This chunk defines `isFabricSupported`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `isFabricSupported`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 118-129
```cpp
  size_t granularity{};
  const auto driver_api = DriverAPI::get();
  C10_CUDA_DRIVER_CHECK(driver_api->cuMemGetAllocationGranularity_(
      &granularity, &prop, CU_MEM_ALLOC_GRANULARITY_RECOMMENDED));

  auto status = driver_api->cuMemCreate_(&handle, granularity, &prop, 0);
  if (status != CUDA_SUCCESS) {
    LOG(INFO)
        << "status " << status
        << " Could not allocate memory with FABRIC handle, falling back to fd handle exchange\n";
    return false;
  }
```
- **EN**: This chunk defines `cuMemCreate_`, which constructs derived state from the current inputs and invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `cuMemCreate_`，其作用是根据当前输入与不变量构建派生状态。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 131-141
```cpp
  // 2. check export
  CUmemFabricHandle sharedHandle;
  status = driver_api->cuMemExportToShareableHandle_(
      &sharedHandle, handle, CU_MEM_HANDLE_TYPE_FABRIC, 0);
  if (status != CUDA_SUCCESS) {
    LOG(INFO)
        << "status " << status
        << " Could not export FABRIC handle, falling back to fd handle exchange\n";
    driver_api->cuMemRelease_(handle);
    return false;
  }
```
- **EN**: This chunk defines `cuMemRelease_`, which implements a focused piece of backend/runtime support logic. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `cuMemRelease_`，其作用是实现一段聚焦的后端/运行时支持逻辑。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 143-159
```cpp
  // 3. check import
  CUmemGenericAllocationHandle import_handle = 0;
  status = driver_api->cuMemImportFromShareableHandle_(
      &import_handle, &sharedHandle, CU_MEM_HANDLE_TYPE_FABRIC);
  if (status != CUDA_SUCCESS) {
    LOG(INFO)
        << "status " << status
        << " Could not import FABRIC handle, falling back to fd handle exchange\n";
    driver_api->cuMemRelease_(handle);
    return false;
  }

  driver_api->cuMemRelease_(import_handle);
  driver_api->cuMemRelease_(handle);
  LOG(INFO) << "using fabric to exchange memory handles\n";
  return true;
}
```
- **EN**: It introduces or extends fabric, which define the main data structures or interfaces for this portion of the file. This chunk defines `cuMemRelease_`, which implements a focused piece of backend/runtime support logic. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 它引入或扩展了 fabric，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `cuMemRelease_`，其作用是实现一段聚焦的后端/运行时支持逻辑。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 161-174
```cpp
#endif
} // namespace

bool get_fabric_access(c10::DeviceIndex dev) {
#if !defined(USE_ROCM) && defined(CUDA_VERSION) && CUDA_VERSION >= 12040 && \
    defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
  TORCH_CHECK(
      num_devices_ >= 0,
      "p2p access cache not initialized. "
      "Ensure c10::cuda::detail::init_p2p_access_cache() is called first.");
  TORCH_CHECK(
      dev >= 0 && dev < num_devices_,
      static_cast<int>(dev),
      " is not a valid device");
```
- **EN**: This chunk defines `defined`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `defined`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 176-190
```cpp
  auto& cache = fabricAccessEnabled_[dev];
  if (cache != -1) {
    return cache;
  }

  auto nvml_device = get_nvml_device(dev);
  if (nvml_device != nullptr) {
    nvmlGpuFabricInfoV_t fabricInfo;
    fabricInfo.state = NVML_GPU_FABRIC_STATE_NOT_SUPPORTED;
    fabricInfo.version = nvmlGpuFabricInfo_v2;
    if (DriverAPI::get()->nvmlDeviceGetGpuFabricInfoV_ == nullptr) {
      cache = 0;
      fabricCliqueId_[dev] = kCliqueIdUnsupported;
      return false;
    }
```
- **EN**: This chunk defines `get_nvml_device`, which manages device or stream context while preserving execution invariants. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `get_nvml_device`，其作用是管理设备或流上下文，同时保持执行不变量。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 191-202
```cpp
    TORCH_CHECK(
        NVML_SUCCESS ==
        DriverAPI::get()->nvmlDeviceGetGpuFabricInfoV_(
            nvml_device, &fabricInfo));
    auto state = fabricInfo.state != NVML_GPU_FABRIC_STATE_NOT_SUPPORTED;
    if (state) {
      fabricCliqueId_[dev] = static_cast<int>(fabricInfo.cliqueId);
      // now perform the full cycle of allocating - exporting - importing memory
      state = isFabricSupported();
    } else {
      fabricCliqueId_[dev] = kCliqueIdUnsupported;
    }
```
- **EN**: This chunk defines `isFabricSupported`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `isFabricSupported`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 203-214
```cpp
    cache = state ? 1 : 0;
    return cache;
  } else {
    cache = 0;
    fabricCliqueId_[dev] = kCliqueIdUnsupported;
    return false;
  }
#else
  (void)dev; // Suppress unused parameter warning
  return false;
#endif
}
```
- **EN**: This chunk continues `isFabricSupported` and expands its control flow, data movement, or edge-case handling. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段延续了 `isFabricSupported`，进一步展开其控制流、数据流转或边界处理逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 216-227
```cpp
int get_fabric_clique_id(c10::DeviceIndex dev) {
#if !defined(USE_ROCM) && defined(CUDA_VERSION) && CUDA_VERSION >= 12040 && \
    defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
  // Ensure cache is populated via get_fabric_access (which does the NVML query
  // and stashes clique_id as a side effect).
  get_fabric_access(dev);
  return fabricCliqueId_[dev];
#else
  (void)dev;
  return kCliqueIdUnsupported;
#endif
}
```
- **EN**: This chunk defines `defined`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `defined`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 229-239
```cpp
std::string get_nvml_fabric_info([[maybe_unused]] c10::DeviceIndex dev) {
#if !defined(USE_ROCM) && defined(CUDA_VERSION) && CUDA_VERSION >= 12040 && \
    defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
  if (DriverAPI::get()->nvmlDeviceGetGpuFabricInfoV_ == nullptr) {
    return "fabric info unsupported (nvmlDeviceGetGpuFabricInfoV not available)";
  }

  auto nvml_device = get_nvml_device(dev);
  if (nvml_device == nullptr) {
    return "fabric info unknown (failed to get NVML device handle)";
  }
```
- **EN**: This chunk defines `get_nvml_device`, which manages device or stream context while preserving execution invariants. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `get_nvml_device`，其作用是管理设备或流上下文，同时保持执行不变量。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 241-257
```cpp
  nvmlGpuFabricInfoV_t info{};
#ifdef nvmlGpuFabricInfo_v3
  bool has_health_summary = false;
  info.version = nvmlGpuFabricInfo_v3;
  if (DriverAPI::get()->nvmlDeviceGetGpuFabricInfoV_(nvml_device, &info) ==
      NVML_SUCCESS) {
    has_health_summary = true;
  } else
#endif
  {
    info = {};
    info.version = nvmlGpuFabricInfo_v2;
    if (DriverAPI::get()->nvmlDeviceGetGpuFabricInfoV_(nvml_device, &info) !=
        NVML_SUCCESS) {
      return "fabric info unknown (nvmlDeviceGetGpuFabricInfoV failed)";
    }
  }
```
- **EN**: This chunk continues `get_nvml_device` and expands its control flow, data movement, or edge-case handling. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段延续了 `get_nvml_device`，进一步展开其控制流、数据流转或边界处理逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 259-276
```cpp
  char uuid_hex[33];
  for (int i = 0; i < 16; ++i) {
    snprintf(uuid_hex + i * 2, 3, "%02x", info.clusterUuid[i]);
  }

  const char* state_str = "unknown";
  switch (info.state) {
    case NVML_GPU_FABRIC_STATE_NOT_SUPPORTED:
      state_str = "not_supported";
      break;
    case NVML_GPU_FABRIC_STATE_NOT_STARTED:
      state_str = "not_started";
      break;
    case NVML_GPU_FABRIC_STATE_IN_PROGRESS:
      state_str = "in_progress";
      break;
    case NVML_GPU_FABRIC_STATE_COMPLETED:
      state_str = "completed";
```
- **EN**: This chunk defines `snprintf`, which implements a focused piece of backend/runtime support logic. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Switch-based dispatch selects specialized behavior for enum variants, backend kinds, or opcode-like cases.
- **CN**: 这一段定义了 `snprintf`，其作用是实现一段聚焦的后端/运行时支持逻辑。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 基于 switch 的分发会为枚举分支、后端类型或类似 opcode 的场景选择专门行为。

### Lines 277-289
```cpp
      break;
  }

  std::ostringstream oss;
  oss << "clique_id=" << info.cliqueId << ", cluster_uuid=" << uuid_hex
      << ", state=" << state_str << ", status=" << info.status
      << ", health_mask=0x" << std::hex << std::setfill('0') << std::setw(8)
      << info.healthMask;
#ifdef nvmlGpuFabricInfo_v3
  if (has_health_summary) {
    oss << ", health_summary=" << std::dec
        << static_cast<int>(info.healthSummary);
  }
```
- **EN**: This chunk defines `static_cast<int>`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 这一段定义了 `static_cast<int>`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 290-297
```cpp
#endif
  return oss.str();
#else
  return "fabric info unsupported (requires CUDA >= 12.4)";
#endif
}

} // namespace c10::cuda
```
- **EN**: This chunk declares `str`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段声明了 `str`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 末尾会返回累积结果，或把更新后的状态交还给调用方。


## Key Concepts / 关键概念
- **CUDA runtime support**
  - EN: Implements CUDA-facing c10 runtime support, including allocators, streams, guards, and device helpers.
  - CN: 实现面向 CUDA 的 c10 运行时支持，包括分配器、流、守卫以及设备辅助逻辑。
- **fabric**
  - EN: `fabric` is one of the dominant symbols declared or implemented in this file.
  - CN: `fabric` 是本文件声明或实现的关键符号之一。
- **init_p2p_access_cache**
  - EN: `init_p2p_access_cache` is one of the dominant symbols declared or implemented in this file.
  - CN: `init_p2p_access_cache` 是本文件声明或实现的关键符号之一。
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
- **Internal includes / 内部依赖**: `c10/cuda/PeerToPeerAccess.h`、`c10/cuda/CUDACachingAllocator.h`、`c10/cuda/CUDAException.h`、`c10/cuda/CUDAGuard.h`、`c10/cuda/driver_api.h`、`c10/util/Exception.h`、`c10/util/Logging.h`、`c10/util/irange.h`
- **Third-party includes / 第三方依赖**: 无
- **Standard includes / 标准库依赖**: `iomanip`、`sstream`、`vector`
- **System includes / 系统依赖**: 无
- **Namespaces / 命名空间**: `c10::cuda`、`detail`
- **Representative symbols / 代表性符号**: `fabric`、`init_p2p_access_cache`、`clear`、`resize`、`get_p2p_access`、`enablePeerAccess`、`defined`、`snprintf`、`isFabricSupported`、`get`
