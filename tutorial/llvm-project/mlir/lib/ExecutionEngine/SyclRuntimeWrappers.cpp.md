# SyclRuntimeWrappers.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/ExecutionEngine/SyclRuntimeWrappers.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements wrappers around the sycl runtime library with C linkage.
  - **CN**: 实现执行引擎运行时、包装层以及面向 JIT 的支持代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- SyclRuntimeWrappers.cpp - MLIR SYCL wrapper library ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-14
```cpp
//
// Implements wrappers around the sycl runtime library with C linkage
//
//===----------------------------------------------------------------------===//

#include <cstdlib>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `cstdlib`.
- **CN**: 引入该编译单元所需的声明，其中包括 `cstdlib`。

### Lines 15-24
```cpp
#include <level_zero/ze_api.h>
#include <sycl/ext/oneapi/backend/level_zero.hpp>
#include <sycl/sycl.hpp>

#ifdef _WIN32
#define SYCL_RUNTIME_EXPORT __declspec(dllexport)
#else
#define SYCL_RUNTIME_EXPORT
#endif // _WIN32

```
- **EN**: Pulls in the declarations needed by this translation unit, including `level_zero/ze_api.h`, `sycl/ext/oneapi/backend/level_zero.hpp`, `sycl/sycl.hpp`.
- **CN**: 引入该编译单元所需的声明，其中包括 `level_zero/ze_api.h`, `sycl/ext/oneapi/backend/level_zero.hpp`, `sycl/sycl.hpp`。

### Lines 25-38
```cpp
namespace {

template <typename F>
auto catchAll(F &&func) {
  try {
    return func();
  } catch (const std::exception &e) {
    fprintf(stderr, "SYCL runtime error: %s\n", e.what());
    fflush(stderr);
    std::exit(EXIT_FAILURE);
  } catch (...) {
    fprintf(stderr, "SYCL runtime error: unknown exception was thrown\n");
    fflush(stderr);
    std::exit(EXIT_FAILURE);
```
- **EN**: Implements logic around `catchAll`, `func`, `fprintf`, `fflush`, and 1 more symbols; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `catchAll`、`func`、`fprintf`、`fflush` 等另外 1 个符号 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 39-51
```cpp
  }
}

#define L0_SAFE_CALL(call)                                                     \
  {                                                                            \
    ze_result_t status = (call);                                               \
    if (status != ZE_RESULT_SUCCESS) {                                         \
      fprintf(stderr, "L0 error %d\n", status);                                \
      fflush(stderr);                                                          \
      abort();                                                                 \
    }                                                                          \
  }

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 52-64
```cpp
} // namespace

static sycl::device getDefaultDevice() {
  static sycl::device syclDevice;
  static bool isDeviceInitialised = false;
  if (!isDeviceInitialised) {
    auto platformList = sycl::platform::get_platforms();
    for (const auto &platform : platformList) {
      auto platformName = platform.get_info<sycl::info::platform::name>();
      bool isLevelZero = platformName.find("Level-Zero") != std::string::npos;
      if (!isLevelZero)
        continue;

```
- **EN**: Implements logic around `getDefaultDevice`, `get_platforms`, `name>`, `find`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `getDefaultDevice`、`get_platforms`、`name>`、`find` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 65-76
```cpp
      syclDevice = platform.get_devices()[0];
      isDeviceInitialised = true;
      return syclDevice;
    }
    throw std::runtime_error(
        "no Level-Zero SYCL platform found; the MLIR SYCL runtime wrapper "
        "currently requires a Level-Zero backend");
  } else {
    return syclDevice;
  }
}

```
- **EN**: Implements logic around `get_devices`, `runtime_error`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `get_devices`、`runtime_error` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 77-90
```cpp
static sycl::context getDefaultContext() {
  static sycl::context syclContext{getDefaultDevice()};
  return syclContext;
}

static void *allocDeviceMemory(sycl::queue *queue, size_t size, bool isShared) {
  void *memPtr = nullptr;
  if (isShared) {
    memPtr = sycl::aligned_alloc_shared(64, size, getDefaultDevice(),
                                        getDefaultContext());
  } else {
    memPtr = sycl::aligned_alloc_device(64, size, *queue);
  }
  if (memPtr == nullptr) {
```
- **EN**: Implements logic around `getDefaultContext`, `getDefaultDevice`, `allocDeviceMemory`, `aligned_alloc_shared`, and 1 more symbols; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `getDefaultContext`、`getDefaultDevice`、`allocDeviceMemory`、`aligned_alloc_shared` 等另外 1 个符号 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 91-99
```cpp
    throw std::runtime_error("mem allocation failed!");
  }
  return memPtr;
}

static void deallocDeviceMemory(sycl::queue *queue, void *ptr) {
  sycl::free(ptr, *queue);
}

```
- **EN**: Implements logic around `runtime_error`, `deallocDeviceMemory`, `free`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `runtime_error`、`deallocDeviceMemory`、`free` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 100-113
```cpp
static ze_module_handle_t loadModule(const void *data, size_t dataSize) {
  assert(data);
  ze_module_handle_t zeModule;
  ze_module_desc_t desc = {ZE_STRUCTURE_TYPE_MODULE_DESC,
                           nullptr,
                           ZE_MODULE_FORMAT_IL_SPIRV,
                           dataSize,
                           (const uint8_t *)data,
                           nullptr,
                           nullptr};
  auto zeDevice = sycl::get_native<sycl::backend::ext_oneapi_level_zero>(
      getDefaultDevice());
  auto zeContext = sycl::get_native<sycl::backend::ext_oneapi_level_zero>(
      getDefaultContext());
```
- **EN**: Implements logic around `loadModule`, `assert`, `ext_oneapi_level_zero>`, `getDefaultDevice`, and 1 more symbols; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `loadModule`、`assert`、`ext_oneapi_level_zero>`、`getDefaultDevice` 等另外 1 个符号 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 114-124
```cpp
  L0_SAFE_CALL(zeModuleCreate(zeContext, zeDevice, &desc, &zeModule, nullptr));
  return zeModule;
}

static sycl::kernel *getKernel(ze_module_handle_t zeModule, const char *name) {
  assert(zeModule);
  assert(name);
  ze_kernel_handle_t zeKernel;
  ze_kernel_desc_t desc = {};
  desc.pKernelName = name;

```
- **EN**: Implements logic around `L0_SAFE_CALL`, `getKernel`, `assert`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `L0_SAFE_CALL`、`getKernel`、`assert` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 125-135
```cpp
  L0_SAFE_CALL(zeKernelCreate(zeModule, &desc, &zeKernel));
  sycl::kernel_bundle<sycl::bundle_state::executable> kernelBundle =
      sycl::make_kernel_bundle<sycl::backend::ext_oneapi_level_zero,
                               sycl::bundle_state::executable>(
          {zeModule}, getDefaultContext());

  auto kernel = sycl::make_kernel<sycl::backend::ext_oneapi_level_zero>(
      {kernelBundle, zeKernel}, getDefaultContext());
  return new sycl::kernel(kernel);
}

```
- **EN**: Implements logic around `L0_SAFE_CALL`, `executable>`, `getDefaultContext`, `ext_oneapi_level_zero>`, and 1 more symbols; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `L0_SAFE_CALL`、`executable>`、`getDefaultContext`、`ext_oneapi_level_zero>` 等另外 1 个符号 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 136-144
```cpp
static void launchKernel(sycl::queue *queue, sycl::kernel *kernel, size_t gridX,
                         size_t gridY, size_t gridZ, size_t blockX,
                         size_t blockY, size_t blockZ, size_t sharedMemBytes,
                         void **params, size_t paramsCount) {
  auto syclGlobalRange =
      sycl::range<3>(blockZ * gridZ, blockY * gridY, blockX * gridX);
  auto syclLocalRange = sycl::range<3>(blockZ, blockY, blockX);
  sycl::nd_range<3> syclNdRange(syclGlobalRange, syclLocalRange);

```
- **EN**: Implements logic around `launchKernel`, `range`, `syclNdRange`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `launchKernel`、`range`、`syclNdRange` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 145-152
```cpp
  queue->submit([&](sycl::handler &cgh) {
    for (size_t i = 0; i < paramsCount; i++) {
      cgh.set_arg(static_cast<uint32_t>(i), *(static_cast<void **>(params[i])));
    }
    cgh.parallel_for(syclNdRange, *kernel);
  });
}

```
- **EN**: Implements logic around `submit`, `set_arg`, `parallel_for`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `submit`、`set_arg`、`parallel_for` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 153-163
```cpp
// Wrappers

extern "C" SYCL_RUNTIME_EXPORT sycl::queue *mgpuStreamCreate() {

  return catchAll([&]() {
    sycl::queue *queue =
        new sycl::queue(getDefaultContext(), getDefaultDevice());
    return queue;
  });
}

```
- **EN**: Implements logic around `mgpuStreamCreate`, `catchAll`, `queue`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `mgpuStreamCreate`、`catchAll`、`queue` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 164-174
```cpp
extern "C" SYCL_RUNTIME_EXPORT void mgpuStreamDestroy(sycl::queue *queue) {
  catchAll([&]() { delete queue; });
}

extern "C" SYCL_RUNTIME_EXPORT void *
mgpuMemAlloc(uint64_t size, sycl::queue *queue, bool isShared) {
  return catchAll([&]() {
    return allocDeviceMemory(queue, static_cast<size_t>(size), true);
  });
}

```
- **EN**: Implements logic around `mgpuStreamDestroy`, `catchAll`, `mgpuMemAlloc`, `allocDeviceMemory`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `mgpuStreamDestroy`、`catchAll`、`mgpuMemAlloc`、`allocDeviceMemory` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 175-182
```cpp
extern "C" SYCL_RUNTIME_EXPORT void mgpuMemFree(void *ptr, sycl::queue *queue) {
  catchAll([&]() {
    if (ptr) {
      deallocDeviceMemory(queue, ptr);
    }
  });
}

```
- **EN**: Implements logic around `mgpuMemFree`, `catchAll`, `deallocDeviceMemory`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `mgpuMemFree`、`catchAll`、`deallocDeviceMemory` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 183-192
```cpp
extern "C" SYCL_RUNTIME_EXPORT ze_module_handle_t
mgpuModuleLoad(const void *data, size_t gpuBlobSize) {
  return catchAll([&]() { return loadModule(data, gpuBlobSize); });
}

extern "C" SYCL_RUNTIME_EXPORT sycl::kernel *
mgpuModuleGetFunction(ze_module_handle_t module, const char *name) {
  return catchAll([&]() { return getKernel(module, name); });
}

```
- **EN**: Implements logic around `mgpuModuleLoad`, `catchAll`, `mgpuModuleGetFunction`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `mgpuModuleLoad`、`catchAll`、`mgpuModuleGetFunction` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 193-203
```cpp
extern "C" SYCL_RUNTIME_EXPORT void
mgpuLaunchKernel(sycl::kernel *kernel, size_t gridX, size_t gridY, size_t gridZ,
                 size_t blockX, size_t blockY, size_t blockZ,
                 size_t sharedMemBytes, sycl::queue *queue, void **params,
                 void ** /*extra*/, size_t paramsCount) {
  return catchAll([&]() {
    launchKernel(queue, kernel, gridX, gridY, gridZ, blockX, blockY, blockZ,
                 sharedMemBytes, params, paramsCount);
  });
}

```
- **EN**: Implements logic around `mgpuLaunchKernel`, `catchAll`, `launchKernel`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `mgpuLaunchKernel`、`catchAll`、`launchKernel` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 204-211
```cpp
extern "C" SYCL_RUNTIME_EXPORT void mgpuStreamSynchronize(sycl::queue *queue) {

  catchAll([&]() { queue->wait(); });
}

extern "C" SYCL_RUNTIME_EXPORT void
mgpuModuleUnload(ze_module_handle_t module) {

```
- **EN**: Implements logic around `mgpuStreamSynchronize`, `catchAll`, `mgpuModuleUnload`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `mgpuStreamSynchronize`、`catchAll`、`mgpuModuleUnload` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 212-218
```cpp
  catchAll([&]() { L0_SAFE_CALL(zeModuleDestroy(module)); });
}

extern "C" SYCL_RUNTIME_EXPORT void
mgpuMemcpy(void *dst, void *src, size_t sizeBytes, sycl::queue *queue) {
  catchAll([&]() { queue->memcpy(dst, src, sizeBytes).wait(); });
}
```
- **EN**: Implements logic around `catchAll`, `mgpuMemcpy`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `catchAll`、`mgpuMemcpy` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

## Key Concepts / 关键概念

- **Execution runtime glue / 执行运行时胶水**:
  - **EN**: Bridges compiled MLIR-generated code with host runtime entry points and wrappers.
  - **CN**: 在由 MLIR 生成的代码与宿主运行时入口点/包装层之间建立桥接。
- **Runtime interop / 运行时互操作**:
  - **EN**: The code exposes or consumes ABI-level hooks used by generated code at runtime.
  - **CN**: 代码暴露或消费运行时由生成代码使用的 ABI 级钩子。

## Dependencies / 依赖关系

- **Standard-library headers / 标准库头文件**: `<cstdlib>`, `<level_zero/ze_api.h>`, `<sycl/ext/oneapi/backend/level_zero.hpp>`, `<sycl/sycl.hpp>`
