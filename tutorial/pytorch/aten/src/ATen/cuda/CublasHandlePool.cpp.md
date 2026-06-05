# CublasHandlePool.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/cuda/CublasHandlePool.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file implements CUDA backend support, with primary focus on `CuBlasLtPoolType`, `CuBlasPoolType`, `hipblasSetWorkspace`.
- 用途（中文）: 该文件实现CUDA 后端支持，核心关注对象是 `CuBlasLtPoolType`, `CuBlasPoolType`, `hipblasSetWorkspace`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```cpp
#include <ATen/cuda/CUDAContext.h>
#include <ATen/cuda/nvrtc_stub/ATenNVRTC.h>
#include <ATen/cuda/detail/DeviceThreadHandles.h>

#include <c10/cuda/CUDACachingAllocator.h>

#include <atomic>
#include <map>
#include <memory>
#include <regex>
#include <shared_mutex>
#include <string>
#include <tuple>

#if defined(USE_ROCM)
#include <rocblas/rocblas.h>
#endif

```
- EN: This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 19-34
```cpp
/**
 * Note [hipblaslt handles]
 * ~~~~~~~~~~~~~~~~~~~~~~~~
 * The cublas documentation states:
 * cuBLAS handle (cublasHandle_t) encapsulates a cuBLASLt handle.
 * Any valid cublasHandle_t can be used in place of cublasLtHandle_t with a simple cast.
 *
 * hipblaslt does not behave in this way.
 * A hipblas handle does not encapsulate a hipblaslt handle.
 *
 * To work around this difference in behavior, a separate handle pool is available for ROCm builds.
 * For CUDA builds, getCurrentCUDABlasLtHandle will alias for getCurrentCUDABlasHandle,
 * whereas for ROCm builds, it is a distinct function.
 *
 * Additionally, hipblaslt cannot share a single handle across multiple streams.
 * On ROCm, getCurrentCUDABlasLtHandle returns a handle unique to each (device, stream)
```
- EN: Focus symbols: `handle`, `each`. This block documents the design intent, assumptions, or usage notes. The comments give maintainers the context needed to interpret the following code correctly.
- CN: 关注符号：`handle`, `each`。该代码块说明设计意图、前提假设或使用说明。这些注释为维护者提供理解后续代码所需的上下文。

### Lines 35-50
```cpp
 * pair, rather than just per-device like the cublas handle pool.
 *
 * The workspace pools are separate for ROCm. On CUDA, the env var
 * TORCH_CUBLASLT_UNIFIED_WORKSPACE can be used to opt-in to unifying the workspace pools.
 */

namespace at::cuda {

namespace {
// -1 means no override; use env var / default
std::atomic<int64_t> cublas_workspace_override{-1};
std::atomic<int64_t> cublaslt_workspace_override{-1};
} // namespace

namespace {

```
- EN: Focus symbols: `at::cuda`, `namespace`. This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 关注符号：`at::cuda`, `namespace`。该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

### Lines 51-68
```cpp
#if defined(USE_ROCM)
void createCublasLtHandle(cublasLtHandle_t *handle) {
  TORCH_CUDABLAS_CHECK(cublasLtCreate(handle));
}

void destroyCublasLtHandle(cublasLtHandle_t handle) {
// this is because of something dumb in the ordering of
// destruction. Sometimes atexit, the cuda context (or something)
// would already be destroyed by the time this gets destroyed. It
// happens in fbcode setting. @colesbury and @soumith decided to not destroy
// the handle as a workaround.
//   - Comments of @soumith copied from cuDNN handle pool implementation
#ifdef NO_CUDNN_DESTROY_HANDLE
#else
    cublasLtDestroy(handle);
#endif
}

```
- EN: Focus symbols: `createCublasLtHandle`, `TORCH_CUDABLAS_CHECK`, `cublasLtCreate`, `destroyCublasLtHandle`, `cublasLtDestroy`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`createCublasLtHandle`, `TORCH_CUDABLAS_CHECK`, `cublasLtCreate`, `destroyCublasLtHandle`, `cublasLtDestroy`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 69-84
```cpp
using CuBlasLtPoolType = DeviceThreadHandlePool<cublasLtHandle_t, createCublasLtHandle, destroyCublasLtHandle>;

// ugly hack until hipblasSetWorkspace exists
static hipblasStatus_t rocBLASStatusToHIPStatus(rocblas_status error) {
    switch(error) {
    case rocblas_status_size_unchanged:
    case rocblas_status_size_increased:
    case rocblas_status_success:
        return HIPBLAS_STATUS_SUCCESS;
    case rocblas_status_invalid_handle:
        return HIPBLAS_STATUS_NOT_INITIALIZED;
    case rocblas_status_not_implemented:
        return HIPBLAS_STATUS_NOT_SUPPORTED;
    case rocblas_status_invalid_pointer:
    case rocblas_status_invalid_size:
    case rocblas_status_invalid_value:
```
- EN: Focus symbols: `CuBlasLtPoolType`, `rocBLASStatusToHIPStatus`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`CuBlasLtPoolType`, `rocBLASStatusToHIPStatus`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 85-102
```cpp
        return HIPBLAS_STATUS_INVALID_VALUE;
    case rocblas_status_memory_error:
        return HIPBLAS_STATUS_ALLOC_FAILED;
    case rocblas_status_internal_error:
        return HIPBLAS_STATUS_INTERNAL_ERROR;
    }
    TORCH_CHECK(false, "HIPBLAS_STATUS_INVALID_ENUM");
}

static hipblasStatus_t hipblasSetWorkspace_replacement(hipblasHandle_t handle, void* addr, size_t size) {
    return rocBLASStatusToHIPStatus(rocblas_set_workspace((rocblas_handle)handle, addr, size));
}

// hipify mappings file correctly maps this but the function doesn't exist yet
#define hipblasSetWorkspace hipblasSetWorkspace_replacement

#endif

```
- EN: Focus symbols: `hipblasSetWorkspace`, `TORCH_CHECK`, `hipblasSetWorkspace_replacement`, `rocBLASStatusToHIPStatus`, `rocblas_set_workspace`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`hipblasSetWorkspace`, `TORCH_CHECK`, `hipblasSetWorkspace_replacement`, `rocBLASStatusToHIPStatus`, `rocblas_set_workspace`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 103-119
```cpp
void createCublasHandle(cublasHandle_t *handle) {
  TORCH_CUDABLAS_CHECK(cublasCreate(handle));
}

void destroyCublasHandle(cublasHandle_t handle) {
// this is because of something dumb in the ordering of
// destruction. Sometimes atexit, the cuda context (or something)
// would already be destroyed by the time this gets destroyed. It
// happens in fbcode setting. @colesbury and @soumith decided to not destroy
// the handle as a workaround.
//   - Comments of @soumith copied from cuDNN handle pool implementation
#ifdef NO_CUDNN_DESTROY_HANDLE
#else
  cublasDestroy(handle);
#endif
}

```
- EN: Focus symbols: `createCublasHandle`, `TORCH_CUDABLAS_CHECK`, `cublasCreate`, `destroyCublasHandle`, `cublasDestroy`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`createCublasHandle`, `TORCH_CUDABLAS_CHECK`, `cublasCreate`, `destroyCublasHandle`, `cublasDestroy`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 120-135
```cpp
using CuBlasPoolType = DeviceThreadHandlePool<cublasHandle_t, createCublasHandle, destroyCublasHandle>;

} // namespace

WorkspaceMapWithMutex& cublas_handle_stream_to_workspace() {
  static auto& instance = *new WorkspaceMapWithMutex;
  return instance;
}

WorkspaceMapWithMutex& cublaslt_handle_stream_to_workspace() {
  static auto& instance = *new WorkspaceMapWithMutex;
  return instance;
}

void clearCublasWorkspaces() {
  {
```
- EN: Focus symbols: `CuBlasPoolType`, `WorkspaceMapWithMutex`, `cublas_handle_stream_to_workspace`, `cublaslt_handle_stream_to_workspace`, `clearCublasWorkspaces`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`CuBlasPoolType`, `WorkspaceMapWithMutex`, `cublas_handle_stream_to_workspace`, `cublaslt_handle_stream_to_workspace`, `clearCublasWorkspaces`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 136-151
```cpp
    auto& workspace = cublas_handle_stream_to_workspace();
    std::unique_lock<std::shared_mutex> lock(workspace.mutex);
    workspace.map.clear();
  }
  {
    auto& workspace = cublaslt_handle_stream_to_workspace();
    std::unique_lock<std::shared_mutex> lock(workspace.mutex);
    workspace.map.clear();
  }
}

void clearCublasWorkspacesForStream(cudaStream_t stream) {
  void* stream_ptr = static_cast<void*>(stream);
  {
    auto& workspace = cublas_handle_stream_to_workspace();
    std::unique_lock<std::shared_mutex> lock(workspace.mutex);
```
- EN: Focus symbols: `cublas_handle_stream_to_workspace`, `lock`, `clear`, `cublaslt_handle_stream_to_workspace`, `clearCublasWorkspacesForStream`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cublas_handle_stream_to_workspace`, `lock`, `clear`, `cublaslt_handle_stream_to_workspace`, `clearCublasWorkspacesForStream`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 152-167
```cpp
    std::erase_if(workspace.map, [stream_ptr](const auto& entry) {
      return std::get<1>(entry.first) == stream_ptr;
    });
  }
  {
    auto& workspace = cublaslt_handle_stream_to_workspace();
    std::unique_lock<std::shared_mutex> lock(workspace.mutex);
    std::erase_if(workspace.map, [stream_ptr](const auto& entry) {
      return std::get<1>(entry.first) == stream_ptr;
    });
  }
}

size_t parseChosenWorkspaceSize() {
  auto val = c10::utils::get_env("CUBLAS_WORKSPACE_CONFIG");
#ifdef USE_ROCM
```
- EN: Focus symbols: `erase_if`, `cublaslt_handle_stream_to_workspace`, `lock`, `parseChosenWorkspaceSize`, `get_env`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`erase_if`, `cublaslt_handle_stream_to_workspace`, `lock`, `parseChosenWorkspaceSize`, `get_env`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 168-184
```cpp
  if (!val) {
    val = c10::utils::get_env("HIPBLAS_WORKSPACE_CONFIG");
  }
  if (!val) {
    // for extra convenience
    val = c10::utils::get_env("ROCBLAS_WORKSPACE_CONFIG");
  }
  /* 32MiB default, 128MiB for gfx94x/gfx95x */
  const bool gfx94_95 = at::detail::getCUDAHooks().isGPUArch({"gfx94", "gfx95"});
  const size_t default_size = gfx94_95 ? 1024 * 128 * 1024 : 1024 * 32 * 1024;
#else
  /* :4096:2:16:8 default, 32MiB for Hopper and Blackwell */
  cudaDeviceProp* properties = at::cuda::getCurrentDeviceProperties();
  const bool use32mb = properties != nullptr && (properties->major == 9 || properties->major == 10 || properties->major == 12);
  const size_t default_size = use32mb ? 4096 * 8 * 1024 : 4096 * 1024 * 2 + 16 * 1024 * 8;
#endif

```
- EN: Focus symbols: `get_env`, `getCUDAHooks`, `isGPUArch`, `getCurrentDeviceProperties`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`get_env`, `getCUDAHooks`, `isGPUArch`, `getCurrentDeviceProperties`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 185-200
```cpp
  if (val) {
    size_t total_size = 0;
    const std::string& config(val.value());
    std::regex exp(":([0-9]+):([0-9]+)");
    std::sregex_iterator next(config.begin(), config.end(), exp);
    std::sregex_iterator end;
    if (next == end) {
      TORCH_WARN("Could not parse CUBLAS_WORKSPACE_CONFIG, using default workspace size of ", default_size, " bytes.");
      return default_size;
    }
    while (next != end) {
      std::smatch match = *next;
      TORCH_CHECK(match.size() == 3, "Expected CUBLAS_WORKSPACE_SPACE_CONFIG match of size 3 (Format :SIZE:COUNT)");
      size_t curr_size = std::stoull(match.str(1));
      size_t count = std::stoull(match.str(2));
      total_size += curr_size * 1024 * count;
```
- EN: Focus symbols: `config`, `value`, `exp`, `next`, `begin`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`config`, `value`, `exp`, `next`, `begin`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 201-216
```cpp
      next++;
    }
    return total_size;
  } else {
    return default_size;
  }
}

#define TORCH_CUBLASLT_UNIFIED_WORKSPACE "TORCH_CUBLASLT_UNIFIED_WORKSPACE"
#ifndef USE_ROCM
inline bool unified_cublas_and_lt_workspaces() {
  static auto unified_env_var = c10::utils::check_env(TORCH_CUBLASLT_UNIFIED_WORKSPACE);
#if !defined(FBCODE)
  static bool unified = (unified_env_var == std::nullopt) || (unified_env_var == true);
#else
  static bool unified = unified_env_var == true;
```
- EN: Focus symbols: `TORCH_CUBLASLT_UNIFIED_WORKSPACE`, `unified_cublas_and_lt_workspaces`, `check_env`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`TORCH_CUBLASLT_UNIFIED_WORKSPACE`, `unified_cublas_and_lt_workspaces`, `check_env`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 217-235
```cpp
#endif
  return unified;
}
#endif

size_t parseCUDABlasLtWorkspaceSize() {
  auto val = c10::utils::get_env("CUBLASLT_WORKSPACE_SIZE");
#ifdef USE_ROCM
  if (!val.has_value()) {
    // accept either env var
    val = c10::utils::get_env("HIPBLASLT_WORKSPACE_SIZE");
  }
  size_t workspace_size = 76*1024; /* Use 76 MB for hipBLASLt */
#else
  /* use CUDABlas default workspace size if unified */
  /* otherwise, use default size in KiB according to #73328 */
  size_t workspace_size = unified_cublas_and_lt_workspaces() ? parseChosenWorkspaceSize() / 1024 : 1024;
#endif

```
- EN: Focus symbols: `parseCUDABlasLtWorkspaceSize`, `get_env`, `has_value`, `unified_cublas_and_lt_workspaces`, `parseChosenWorkspaceSize`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`parseCUDABlasLtWorkspaceSize`, `get_env`, `has_value`, `unified_cublas_and_lt_workspaces`, `parseChosenWorkspaceSize`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 236-255
```cpp
  if (val.has_value()) {
    try {
      workspace_size = std::stoi(val.value());
    } catch (std::invalid_argument const&) {
      TORCH_WARN(
          "invalid CUBLASLT_WORKSPACE_SIZE,",
          " using default workspace size of ",
          workspace_size,
          " KiB.");
    } catch (std::out_of_range const&) {
      TORCH_WARN(
          "CUBLASLT_WORKSPACE_SIZE out of range,",
          " using default workspace size of ",
          workspace_size,
          " KiB.");
    }
  }
  return workspace_size * 1024;
}

```
- EN: Focus symbols: `has_value`, `stoi`, `value`, `TORCH_WARN`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`has_value`, `stoi`, `value`, `TORCH_WARN`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 256-272
```cpp
size_t getChosenWorkspaceSize() {
  int64_t ov = cublas_workspace_override.load(std::memory_order_relaxed);
  if (ov >= 0) {
    return static_cast<size_t>(ov);
  }
  static size_t pool_size = parseChosenWorkspaceSize();
  return pool_size;
}

void setChosenWorkspaceSize(size_t size) {
  cublas_workspace_override.store(static_cast<int64_t>(size), std::memory_order_relaxed);
}

void setCUDABlasLtWorkspaceSize(size_t size) {
  cublaslt_workspace_override.store(static_cast<int64_t>(size), std::memory_order_relaxed);
}

```
- EN: Focus symbols: `getChosenWorkspaceSize`, `load`, `parseChosenWorkspaceSize`, `setChosenWorkspaceSize`, `store`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getChosenWorkspaceSize`, `load`, `parseChosenWorkspaceSize`, `setChosenWorkspaceSize`, `store`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 273-288
```cpp
void resetChosenWorkspaceSize() {
  cublas_workspace_override.store(-1, std::memory_order_relaxed);
}

void resetCUDABlasLtWorkspaceSize() {
  cublaslt_workspace_override.store(-1, std::memory_order_relaxed);
}

size_t getCUDABlasLtWorkspaceSize() {
  int64_t ov = cublaslt_workspace_override.load(std::memory_order_relaxed);
  const size_t pool_size = [&] {
    if (ov >= 0) {
      return static_cast<size_t>(ov);
    }
    static size_t parsed_pool_size = parseCUDABlasLtWorkspaceSize();
    return parsed_pool_size;
```
- EN: Focus symbols: `resetChosenWorkspaceSize`, `store`, `resetCUDABlasLtWorkspaceSize`, `getCUDABlasLtWorkspaceSize`, `load`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`resetChosenWorkspaceSize`, `store`, `resetCUDABlasLtWorkspaceSize`, `getCUDABlasLtWorkspaceSize`, `load`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 289-306
```cpp
  }();
#ifndef USE_ROCM
  if (unified_cublas_and_lt_workspaces()) {
    size_t cublasWorkspaceSize = getChosenWorkspaceSize();
    if (cublasWorkspaceSize < pool_size) {
      TORCH_WARN_ONCE("Requested unified CUBLASLT workspace size of ", pool_size,
                      " bytes exceeds CUBLAS workspace size of ", cublasWorkspaceSize,
                      " bytes. Please increase CUBLAS workspace size",
                      " via CUBLAS_WORKSPACE_CONFIG or decrease requested"
                      " CUBLASLT_WORKSPACE_SIZE. Otherwise CUBLASLT workspace"
                      " size will be limited to the CUBLAS workspace size.");
      return cublasWorkspaceSize;
    }
  }
#endif
  return pool_size;
}

```
- EN: Focus symbols: `unified_cublas_and_lt_workspaces`, `getChosenWorkspaceSize`, `TORCH_WARN_ONCE`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`unified_cublas_and_lt_workspaces`, `getChosenWorkspaceSize`, `TORCH_WARN_ONCE`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 307-322
```cpp
at::DataPtr getNewWorkspace() {
  return c10::cuda::CUDACachingAllocator::get()->allocate(getChosenWorkspaceSize());
}

at::DataPtr getNewCUDABlasLtWorkspace() {
  return c10::cuda::CUDACachingAllocator::get()->allocate(getCUDABlasLtWorkspaceSize());
}

void setWorkspaceForHandle(cublasHandle_t handle, c10::cuda::CUDAStream stream) {
  cudaStream_t _stream = stream;
  auto key = std::make_tuple(static_cast<void *>(handle), static_cast<void *>(_stream));

  auto& workspace = cublas_handle_stream_to_workspace();

  size_t workspace_size = getChosenWorkspaceSize();

```
- EN: Focus symbols: `getNewWorkspace`, `get`, `allocate`, `getChosenWorkspaceSize`, `getNewCUDABlasLtWorkspace`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getNewWorkspace`, `get`, `allocate`, `getChosenWorkspaceSize`, `getNewCUDABlasLtWorkspace`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 323-338
```cpp
  // Fast path: check if workspace already exists and is large enough
  {
    std::shared_lock<std::shared_mutex> lock(workspace.mutex);
    auto workspace_it = workspace.map.find(key);
    if (workspace_it != workspace.map.end() && workspace_it->second.second >= workspace_size) {
      TORCH_CUDABLAS_CHECK(cublasSetWorkspace(
          handle, workspace_it->second.first.get(), workspace_size));
      return;
    }
  }

  // Slow path: allocate workspace outside the lock
  auto new_workspace = getNewWorkspace();

  // Insert with lock, replacing any undersized entry
  {
```
- EN: Focus symbols: `lock`, `find`, `end`, `TORCH_CUDABLAS_CHECK`, `cublasSetWorkspace`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`lock`, `find`, `end`, `TORCH_CUDABLAS_CHECK`, `cublasSetWorkspace`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 339-354
```cpp
    std::unique_lock<std::shared_mutex> lock(workspace.mutex);
    workspace.map.insert_or_assign(key, std::make_pair(std::move(new_workspace), workspace_size));
    auto workspace_it = workspace.map.find(key);
    TORCH_CUDABLAS_CHECK(
        cublasSetWorkspace(handle, workspace_it->second.first.get(), workspace_size));
  }
}

void* getCUDABlasLtWorkspace() {
#ifndef USE_ROCM
  if (unified_cublas_and_lt_workspaces()) {
    cublasHandle_t handle = at::cuda::getCurrentCUDABlasHandle(/*setup=*/false);
    auto stream = c10::cuda::getCurrentCUDAStream();
    cudaStream_t _stream = stream;
    auto key = std::make_tuple(static_cast<void *>(handle), static_cast<void *>(_stream));
    auto& workspace = at::cuda::cublas_handle_stream_to_workspace();
```
- EN: Focus symbols: `lock`, `insert_or_assign`, `make_pair`, `move`, `find`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`lock`, `insert_or_assign`, `make_pair`, `move`, `find`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 355-370
```cpp
    {
      std::shared_lock<std::shared_mutex> lock(workspace.mutex);
      auto workspace_it = workspace.map.find(key);
      if (workspace_it != workspace.map.end()) {
        return workspace_it->second.first.mutable_get();
      }
    }
    // First use for this handle+stream pair — allocate and insert directly.
    // No need to call cublasSetWorkspace; Lt passes workspace explicitly.
    auto new_workspace = getNewWorkspace();
    {
      std::unique_lock<std::shared_mutex> lock(workspace.mutex);
      auto workspace_it = workspace.map.try_emplace(key, std::make_pair(std::move(new_workspace), getChosenWorkspaceSize())).first;
      return workspace_it->second.first.mutable_get();
    }
  }
```
- EN: Focus symbols: `lock`, `find`, `end`, `mutable_get`, `getNewWorkspace`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`lock`, `find`, `end`, `mutable_get`, `getNewWorkspace`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 371-389
```cpp
#endif
  cublasLtHandle_t handle = getCurrentCUDABlasLtHandle();
  auto stream = c10::cuda::getCurrentCUDAStream();
  cudaStream_t _stream = stream;
  auto key = std::make_tuple(static_cast<void *>(handle), static_cast<void *>(_stream));

  auto& workspace = cublaslt_handle_stream_to_workspace();

  size_t workspace_size = getCUDABlasLtWorkspaceSize();

  // Fast path: check if workspace already exists and is large enough
  {
    std::shared_lock<std::shared_mutex> lock(workspace.mutex);
    auto workspace_it = workspace.map.find(key);
    if (workspace_it != workspace.map.end() && workspace_it->second.second >= workspace_size) {
      return workspace_it->second.first.mutable_get();
    }
  }

```
- EN: Focus symbols: `getCurrentCUDABlasLtHandle`, `getCurrentCUDAStream`, `make_tuple`, `cublaslt_handle_stream_to_workspace`, `getCUDABlasLtWorkspaceSize`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getCurrentCUDABlasLtHandle`, `getCurrentCUDAStream`, `make_tuple`, `cublaslt_handle_stream_to_workspace`, `getCUDABlasLtWorkspaceSize`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 390-405
```cpp
  // Slow path: allocate workspace outside the lock
  auto new_workspace = getNewCUDABlasLtWorkspace();

  // Insert with lock, replacing any undersized entry
  {
    std::unique_lock<std::shared_mutex> lock(workspace.mutex);
    workspace.map.insert_or_assign(key, std::make_pair(std::move(new_workspace), workspace_size));
    auto workspace_it = workspace.map.find(key);
    return workspace_it->second.first.mutable_get();
  }
}

cublasHandle_t getCurrentCUDABlasHandle(bool setup) {
  c10::DeviceIndex device = 0;
  AT_CUDA_CHECK(c10::cuda::GetDevice(&device));

```
- EN: Focus symbols: `getNewCUDABlasLtWorkspace`, `lock`, `insert_or_assign`, `make_pair`, `move`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getNewCUDABlasLtWorkspace`, `lock`, `insert_or_assign`, `make_pair`, `move`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 406-421
```cpp
#if !defined(USE_ROCM)
  CUcontext pctx = nullptr;
  at::globalContext().getNVRTC().cuCtxGetCurrent(&pctx);
  if (C10_UNLIKELY(!pctx)) {
    TORCH_WARN_ONCE("Attempting to run cuBLAS, but there was no current CUDA context! Attempting to set the primary context...");
    at::globalContext().getNVRTC().cuDevicePrimaryCtxRetain(&pctx, device);
    at::globalContext().getNVRTC().cuCtxSetCurrent(pctx);
  }
#endif

  // Thread local PoolWindows are lazily-initialized
  // to avoid initialization issues that caused hangs on Windows.
  // See: https://github.com/pytorch/pytorch/pull/22405
  // This thread local unique_ptrs will be destroyed when the thread terminates,
  // releasing its reserved handles back to the pool.

```
- EN: Focus symbols: `globalContext`, `getNVRTC`, `cuCtxGetCurrent`, `C10_UNLIKELY`, `TORCH_WARN_ONCE`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`globalContext`, `getNVRTC`, `cuCtxGetCurrent`, `C10_UNLIKELY`, `TORCH_WARN_ONCE`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 422-437
```cpp
  // Use a leaky singleton for the pool following standard practice around
  // singletons: https://isocpp.org/wiki/faq/ctors#construct-on-first-use-v2
  static auto pool = std::shared_ptr<CuBlasPoolType>(
      new CuBlasPoolType(), [](CuBlasPoolType* p) {
        // Leak the memory.
      });
  thread_local std::unique_ptr<CuBlasPoolType::PoolWindow> myPoolWindow(
      pool->newPoolWindow());

  auto handle = myPoolWindow->reserve(device);

  if (!setup) {
    return handle;
  }

  auto stream = c10::cuda::getCurrentCUDAStream();
```
- EN: Focus symbols: `CuBlasPoolType`, `myPoolWindow`, `newPoolWindow`, `reserve`, `getCurrentCUDAStream`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`CuBlasPoolType`, `myPoolWindow`, `newPoolWindow`, `reserve`, `getCurrentCUDAStream`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 438-453
```cpp
  TORCH_CUDABLAS_CHECK(cublasSetStream(handle, stream));
  // We explicitly set the cublas workspace even though CUDA 12.2+ fixed the
  // issue where memory usage increased during graph capture.
  // original issue: https://github.com/pytorch/pytorch/pull/83461
  // This is because in CUDA 12.2+, the use of cudaMallocAsync in cublas
  // will allocate memory dynamically (even if they're cheap) outside
  // PyTorch's CUDA caching allocator. It's possible that CCA used up
  // all the memory and cublas's cudaMallocAsync will return OOM
  setWorkspaceForHandle(handle, stream);

#if !defined(USE_ROCM)
  // On CUDA >= 11, and architecture >= Ampere, cuBLAS can use TF32 to speedup
  // FP32 data type calculations based on the value of the allow_tf32 flag.
  // To enable TF32, set the math mode of the handle to CUBLAS_TF32_TENSOR_OP_MATH.
  if (!NoTF32Guard::should_disable_tf32() &&
      at::globalContext().float32Precision(at::Float32Backend::CUDA, at::Float32Op::MATMUL) == at::Float32Precision::TF32) {
```
- EN: Focus symbols: `TORCH_CUDABLAS_CHECK`, `cublasSetStream`, `setWorkspaceForHandle`, `should_disable_tf32`, `globalContext`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`TORCH_CUDABLAS_CHECK`, `cublasSetStream`, `setWorkspaceForHandle`, `should_disable_tf32`, `globalContext`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 454-469
```cpp
    TORCH_CUDABLAS_CHECK(cublasSetMathMode(handle, CUBLAS_TF32_TENSOR_OP_MATH));
  } else {
    TORCH_CUDABLAS_CHECK(cublasSetMathMode(handle, CUBLAS_DEFAULT_MATH));
  }
#else
  hipblasAtomicsMode_t hipblas_mode;
  if (at::globalContext().deterministicAlgorithms()) {
    hipblas_mode = HIPBLAS_ATOMICS_NOT_ALLOWED;
  } else {
    hipblas_mode = HIPBLAS_ATOMICS_ALLOWED;
  }
  TORCH_CUDABLAS_CHECK(hipblasSetAtomicsMode(handle, hipblas_mode));
#endif
  return handle;
}

```
- EN: Focus symbols: `TORCH_CUDABLAS_CHECK`, `cublasSetMathMode`, `globalContext`, `deterministicAlgorithms`, `hipblasSetAtomicsMode`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`TORCH_CUDABLAS_CHECK`, `cublasSetMathMode`, `globalContext`, `deterministicAlgorithms`, `hipblasSetAtomicsMode`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 470-489
```cpp
cublasLtHandle_t getCurrentCUDABlasLtHandle() {
#ifdef USE_ROCM
  c10::DeviceIndex device = 0;
  AT_CUDA_CHECK(c10::cuda::GetDevice(&device));

  // Thread local PoolWindows are lazily-initialized
  // to avoid initialization issues that caused hangs on Windows.
  // See: https://github.com/pytorch/pytorch/pull/22405
  // This thread local unique_ptrs will be destroyed when the thread terminates,
  // releasing its reserved handles back to the pool.

  // Use a leaky singleton for the pool following standard practice around
  // singletons: https://isocpp.org/wiki/faq/ctors#construct-on-first-use-v2
  static auto pool = std::shared_ptr<CuBlasLtPoolType>(
      new CuBlasLtPoolType(), [](CuBlasLtPoolType* p) {
        // Leak the memory.
      });
  thread_local std::unique_ptr<CuBlasLtPoolType::PoolWindow> myPoolWindow(
      pool->newPoolWindow());

```
- EN: Focus symbols: `getCurrentCUDABlasLtHandle`, `AT_CUDA_CHECK`, `GetDevice`, `CuBlasLtPoolType`, `myPoolWindow`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getCurrentCUDABlasLtHandle`, `AT_CUDA_CHECK`, `GetDevice`, `CuBlasLtPoolType`, `myPoolWindow`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 490-501
```cpp
  // hipblaslt cannot share a single handle across multiple streams,
  // so reserve a handle unique to each (device, stream) pair.
  auto stream = c10::cuda::getCurrentCUDAStream();
  cudaStream_t _stream = stream;
  auto handle = myPoolWindow->reserve(device, static_cast<void*>(_stream));
  return handle;
#else
  return reinterpret_cast<cublasLtHandle_t>(getCurrentCUDABlasHandle(/*setup=*/false));
#endif
}

} // namespace at::cuda
```
- EN: Focus symbols: `at::cuda`, `getCurrentCUDAStream`, `reserve`, `getCurrentCUDABlasHandle`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`at::cuda`, `getCurrentCUDAStream`, `reserve`, `getCurrentCUDABlasHandle`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

## Key Concepts / 关键概念
- CUDA backend support / CUDA 后端支持
- Conditional compilation / 条件编译
- Runtime validation / 运行时校验
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/cuda/CUDAContext.h`, `ATen/cuda/nvrtc_stub/ATenNVRTC.h`, `ATen/cuda/detail/DeviceThreadHandles.h`, `c10/cuda/CUDACachingAllocator.h`
- External/system includes / 外部或系统头: `atomic`, `map`, `memory`, `regex`, `shared_mutex`, `string`, `tuple`, `rocblas/rocblas.h`
- Inferred semantic dependencies / 推断出的语义依赖: runtime validation / 运行时校验; namespace scoping / 命名空间作用域
