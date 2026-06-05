# VulkanRuntimeWrappers.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/ExecutionEngine/VulkanRuntimeWrappers.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements C runtime wrappers around the VulkanRuntime.
  - **CN**: 实现执行引擎运行时、包装层以及面向 JIT 的支持代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- VulkanRuntimeWrappers.cpp - MLIR Vulkan runner wrapper library -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-18
```cpp
//
// Implements C runtime wrappers around the VulkanRuntime.
//
//===----------------------------------------------------------------------===//

#include <iostream>
#include <mutex>
#include <numeric>
#include <string>
#include <vector>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `iostream`, `mutex`, `numeric`, `string`.
- **CN**: 引入该编译单元所需的声明，其中包括 `iostream`, `mutex`, `numeric`, `string`。

### Lines 19-28
```cpp
#include "VulkanRuntime.h"

// Explicitly export entry points to the vulkan-runtime-wrapper.

#ifdef _WIN32
#define VULKAN_WRAPPER_SYMBOL_EXPORT __declspec(dllexport)
#else
#define VULKAN_WRAPPER_SYMBOL_EXPORT __attribute__((visibility("default")))
#endif // _WIN32

```
- **EN**: Pulls in the declarations needed by this translation unit, including `VulkanRuntime.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `VulkanRuntime.h`。

### Lines 29-37
```cpp
namespace {

class VulkanModule;

// Class to be a thing that can be returned from `mgpuModuleGetFunction`.
struct VulkanFunction {
  VulkanModule *module;
  std::string name;

```
- **EN**: Introduces declarations for `VulkanModule`, `VulkanFunction`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `VulkanModule`、`VulkanFunction` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 38-44
```cpp
  VulkanFunction(VulkanModule *module, const char *name)
      : module(module), name(name) {}
};

// Class to own a copy of the SPIR-V provided to `mgpuModuleLoad` and to manage
// allocation of pointers returned from `mgpuModuleGetFunction`.
class VulkanModule {
```
- **EN**: Introduces declarations for `VulkanModule`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `VulkanModule` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 45-54
```cpp
public:
  VulkanModule(const uint8_t *ptr, size_t sizeInBytes)
      : blob(ptr, ptr + sizeInBytes) {}
  ~VulkanModule() = default;

  VulkanFunction *getFunction(const char *name) {
    return functions.emplace_back(std::make_unique<VulkanFunction>(this, name))
        .get();
  }

```
- **EN**: Implements logic around `VulkanModule`, `blob`, `~VulkanModule`, `getFunction`, and 2 more symbols; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `VulkanModule`、`blob`、`~VulkanModule`、`getFunction` 等另外 2 个符号 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 55-62
```cpp
  uint8_t *blobData() { return blob.data(); }
  size_t blobSizeInBytes() const { return blob.size(); }

private:
  std::vector<uint8_t> blob;
  std::vector<std::unique_ptr<VulkanFunction>> functions;
};

```
- **EN**: Implements logic around `blobData`, `blobSizeInBytes`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `blobData`、`blobSizeInBytes` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 63-69
```cpp
class VulkanRuntimeManager {
public:
  VulkanRuntimeManager() = default;
  VulkanRuntimeManager(const VulkanRuntimeManager &) = delete;
  VulkanRuntimeManager operator=(const VulkanRuntimeManager &) = delete;
  ~VulkanRuntimeManager() = default;

```
- **EN**: Introduces declarations for `VulkanRuntimeManager`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `VulkanRuntimeManager` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 70-80
```cpp
  void setResourceData(DescriptorSetIndex setIndex, BindingIndex bindIndex,
                       const VulkanHostMemoryBuffer &memBuffer) {
    std::lock_guard<std::mutex> lock(mutex);
    vulkanRuntime.setResourceData(setIndex, bindIndex, memBuffer);
  }

  void setEntryPoint(const char *entryPoint) {
    std::lock_guard<std::mutex> lock(mutex);
    vulkanRuntime.setEntryPoint(entryPoint);
  }

```
- **EN**: Implements logic around `setResourceData`, `lock`, `setEntryPoint`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `setResourceData`、`lock`、`setEntryPoint` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 81-90
```cpp
  void setNumWorkGroups(NumWorkGroups numWorkGroups) {
    std::lock_guard<std::mutex> lock(mutex);
    vulkanRuntime.setNumWorkGroups(numWorkGroups);
  }

  void setShaderModule(uint8_t *shader, uint32_t size) {
    std::lock_guard<std::mutex> lock(mutex);
    vulkanRuntime.setShaderModule(shader, size);
  }

```
- **EN**: Implements logic around `setNumWorkGroups`, `lock`, `setShaderModule`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `setNumWorkGroups`、`lock`、`setShaderModule` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 91-99
```cpp
  void runOnVulkan() {
    std::lock_guard<std::mutex> lock(mutex);
    if (failed(vulkanRuntime.initRuntime()) || failed(vulkanRuntime.run()) ||
        failed(vulkanRuntime.updateHostMemoryBuffers()) ||
        failed(vulkanRuntime.destroy())) {
      std::cerr << "runOnVulkan failed";
    }
  }

```
- **EN**: Implements logic around `runOnVulkan`, `lock`, `failed`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `runOnVulkan`、`lock`、`failed` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 100-106
```cpp
private:
  VulkanRuntime vulkanRuntime;
  std::mutex mutex;
};

} // namespace

```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 107-115
```cpp
template <typename T, int N>
struct MemRefDescriptor {
  T *allocated;
  T *aligned;
  int64_t offset;
  int64_t sizes[N];
  int64_t strides[N];
};

```
- **EN**: Introduces declarations for `MemRefDescriptor`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `MemRefDescriptor` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 116-123
```cpp
extern "C" {

//===----------------------------------------------------------------------===//
//
// Wrappers intended for mlir-runner. Uses of GPU dialect operations get
// lowered to calls to these functions by GPUToLLVMConversionPass.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 124-132
```cpp

VULKAN_WRAPPER_SYMBOL_EXPORT void *mgpuStreamCreate() {
  return new VulkanRuntimeManager();
}

VULKAN_WRAPPER_SYMBOL_EXPORT void mgpuStreamDestroy(void *vkRuntimeManager) {
  delete static_cast<VulkanRuntimeManager *>(vkRuntimeManager);
}

```
- **EN**: Implements logic around `mgpuStreamCreate`, `VulkanRuntimeManager`, `mgpuStreamDestroy`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `mgpuStreamCreate`、`VulkanRuntimeManager`、`mgpuStreamDestroy` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 133-142
```cpp
VULKAN_WRAPPER_SYMBOL_EXPORT void mgpuStreamSynchronize(void *) {
  // Currently a no-op as the other operations are synchronous.
}

VULKAN_WRAPPER_SYMBOL_EXPORT void *mgpuModuleLoad(const void *data,
                                                  size_t gpuBlobSize) {
  // gpuBlobSize is the size of the data in bytes.
  return new VulkanModule(static_cast<const uint8_t *>(data), gpuBlobSize);
}

```
- **EN**: Implements logic around `mgpuStreamSynchronize`, `mgpuModuleLoad`, `VulkanModule`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `mgpuStreamSynchronize`、`mgpuModuleLoad`、`VulkanModule` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 143-153
```cpp
VULKAN_WRAPPER_SYMBOL_EXPORT void mgpuModuleUnload(void *vkModule) {
  delete static_cast<VulkanModule *>(vkModule);
}

VULKAN_WRAPPER_SYMBOL_EXPORT void *mgpuModuleGetFunction(void *vkModule,
                                                         const char *name) {
  if (!vkModule)
    abort();
  return static_cast<VulkanModule *>(vkModule)->getFunction(name);
}

```
- **EN**: Implements logic around `mgpuModuleUnload`, `mgpuModuleGetFunction`, `abort`, `getFunction`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `mgpuModuleUnload`、`mgpuModuleGetFunction`、`abort`、`getFunction` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 154-160
```cpp
VULKAN_WRAPPER_SYMBOL_EXPORT void
mgpuLaunchKernel(void *vkKernel, size_t gridX, size_t gridY, size_t gridZ,
                 size_t /*blockX*/, size_t /*blockY*/, size_t /*blockZ*/,
                 size_t /*smem*/, void *vkRuntimeManager, void **params,
                 void ** /*extra*/, size_t paramsCount) {
  auto *manager = static_cast<VulkanRuntimeManager *>(vkRuntimeManager);

```
- **EN**: Implements logic around `mgpuLaunchKernel`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `mgpuLaunchKernel` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 161-174
```cpp
  // GpuToLLVMConversionPass with the kernelBarePtrCallConv and
  // kernelIntersperseSizeCallConv options will set up the params array like:
  // { &memref_ptr0, &memref_size0, &memref_ptr1, &memref_size1, ... }
  const size_t paramsPerMemRef = 2;
  if (paramsCount % paramsPerMemRef != 0) {
    abort(); // This would indicate a serious calling convention mismatch.
  }
  const DescriptorSetIndex setIndex = 0;
  BindingIndex bindIndex = 0;
  for (size_t i = 0; i < paramsCount; i += paramsPerMemRef) {
    void *memrefBufferBasePtr = *static_cast<void **>(params[i + 0]);
    size_t memrefBufferSize = *static_cast<size_t *>(params[i + 1]);
    VulkanHostMemoryBuffer memBuffer{memrefBufferBasePtr,
                                     static_cast<uint32_t>(memrefBufferSize)};
```
- **EN**: Implements logic around `abort`, `static_cast`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `abort`、`static_cast` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 175-182
```cpp
    manager->setResourceData(setIndex, bindIndex, memBuffer);
    ++bindIndex;
  }

  manager->setNumWorkGroups(NumWorkGroups{static_cast<uint32_t>(gridX),
                                          static_cast<uint32_t>(gridY),
                                          static_cast<uint32_t>(gridZ)});

```
- **EN**: Implements logic around `setResourceData`, `setNumWorkGroups`, `static_cast`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `setResourceData`、`setNumWorkGroups`、`static_cast` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 183-189
```cpp
  auto *function = static_cast<VulkanFunction *>(vkKernel);
  // Expected size should be in bytes.
  manager->setShaderModule(
      function->module->blobData(),
      static_cast<uint32_t>(function->module->blobSizeInBytes()));
  manager->setEntryPoint(function->name.c_str());

```
- **EN**: Implements logic around `setShaderModule`, `blobData`, `static_cast`, `setEntryPoint`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `setShaderModule`、`blobData`、`static_cast`、`setEntryPoint` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 190-197
```cpp
  manager->runOnVulkan();
}

//===----------------------------------------------------------------------===//
//
// Miscellaneous utility functions that can be directly used by tests.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `runOnVulkan`; this block bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `runOnVulkan` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 198-205
```cpp

/// Fills the given 1D float memref with the given float value.
VULKAN_WRAPPER_SYMBOL_EXPORT void
_mlir_ciface_fillResource1DFloat(MemRefDescriptor<float, 1> *ptr, // NOLINT
                                 float value) {
  std::fill_n(ptr->allocated, ptr->sizes[0], value);
}

```
- **EN**: Implements logic around `_mlir_ciface_fillResource1DFloat`, `fill_n`; this block manipulates core MLIR IR objects; bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `_mlir_ciface_fillResource1DFloat`、`fill_n` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 206-212
```cpp
/// Fills the given 2D float memref with the given float value.
VULKAN_WRAPPER_SYMBOL_EXPORT void
_mlir_ciface_fillResource2DFloat(MemRefDescriptor<float, 2> *ptr, // NOLINT
                                 float value) {
  std::fill_n(ptr->allocated, ptr->sizes[0] * ptr->sizes[1], value);
}

```
- **EN**: Implements logic around `_mlir_ciface_fillResource2DFloat`, `fill_n`; this block manipulates core MLIR IR objects; bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `_mlir_ciface_fillResource2DFloat`、`fill_n` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 213-220
```cpp
/// Fills the given 3D float memref with the given float value.
VULKAN_WRAPPER_SYMBOL_EXPORT void
_mlir_ciface_fillResource3DFloat(MemRefDescriptor<float, 3> *ptr, // NOLINT
                                 float value) {
  std::fill_n(ptr->allocated, ptr->sizes[0] * ptr->sizes[1] * ptr->sizes[2],
              value);
}

```
- **EN**: Implements logic around `_mlir_ciface_fillResource3DFloat`, `fill_n`; this block manipulates core MLIR IR objects; bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `_mlir_ciface_fillResource3DFloat`、`fill_n` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 221-227
```cpp
/// Fills the given 1D int memref with the given int value.
VULKAN_WRAPPER_SYMBOL_EXPORT void
_mlir_ciface_fillResource1DInt(MemRefDescriptor<int32_t, 1> *ptr, // NOLINT
                               int32_t value) {
  std::fill_n(ptr->allocated, ptr->sizes[0], value);
}

```
- **EN**: Implements logic around `_mlir_ciface_fillResource1DInt`, `fill_n`; this block manipulates core MLIR IR objects; bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `_mlir_ciface_fillResource1DInt`、`fill_n` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 228-234
```cpp
/// Fills the given 2D int memref with the given int value.
VULKAN_WRAPPER_SYMBOL_EXPORT void
_mlir_ciface_fillResource2DInt(MemRefDescriptor<int32_t, 2> *ptr, // NOLINT
                               int32_t value) {
  std::fill_n(ptr->allocated, ptr->sizes[0] * ptr->sizes[1], value);
}

```
- **EN**: Implements logic around `_mlir_ciface_fillResource2DInt`, `fill_n`; this block manipulates core MLIR IR objects; bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `_mlir_ciface_fillResource2DInt`、`fill_n` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 235-242
```cpp
/// Fills the given 3D int memref with the given int value.
VULKAN_WRAPPER_SYMBOL_EXPORT void
_mlir_ciface_fillResource3DInt(MemRefDescriptor<int32_t, 3> *ptr, // NOLINT
                               int32_t value) {
  std::fill_n(ptr->allocated, ptr->sizes[0] * ptr->sizes[1] * ptr->sizes[2],
              value);
}

```
- **EN**: Implements logic around `_mlir_ciface_fillResource3DInt`, `fill_n`; this block manipulates core MLIR IR objects; bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `_mlir_ciface_fillResource3DInt`、`fill_n` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 243-249
```cpp
/// Fills the given 1D int memref with the given int8 value.
VULKAN_WRAPPER_SYMBOL_EXPORT void
_mlir_ciface_fillResource1DInt8(MemRefDescriptor<int8_t, 1> *ptr, // NOLINT
                                int8_t value) {
  std::fill_n(ptr->allocated, ptr->sizes[0], value);
}

```
- **EN**: Implements logic around `_mlir_ciface_fillResource1DInt8`, `fill_n`; this block manipulates core MLIR IR objects; bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `_mlir_ciface_fillResource1DInt8`、`fill_n` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 250-256
```cpp
/// Fills the given 2D int memref with the given int8 value.
VULKAN_WRAPPER_SYMBOL_EXPORT void
_mlir_ciface_fillResource2DInt8(MemRefDescriptor<int8_t, 2> *ptr, // NOLINT
                                int8_t value) {
  std::fill_n(ptr->allocated, ptr->sizes[0] * ptr->sizes[1], value);
}

```
- **EN**: Implements logic around `_mlir_ciface_fillResource2DInt8`, `fill_n`; this block manipulates core MLIR IR objects; bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `_mlir_ciface_fillResource2DInt8`、`fill_n` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 257-264
```cpp
/// Fills the given 3D int memref with the given int8 value.
VULKAN_WRAPPER_SYMBOL_EXPORT void
_mlir_ciface_fillResource3DInt8(MemRefDescriptor<int8_t, 3> *ptr, // NOLINT
                                int8_t value) {
  std::fill_n(ptr->allocated, ptr->sizes[0] * ptr->sizes[1] * ptr->sizes[2],
              value);
}
}
```
- **EN**: Implements logic around `_mlir_ciface_fillResource3DInt8`, `fill_n`; this block manipulates core MLIR IR objects; bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `_mlir_ciface_fillResource3DInt8`、`fill_n` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并在 MLIR 生成代码与运行时入口点之间建立桥接。

## Key Concepts / 关键概念

- **Execution runtime glue / 执行运行时胶水**:
  - **EN**: Bridges compiled MLIR-generated code with host runtime entry points and wrappers.
  - **CN**: 在由 MLIR 生成的代码与宿主运行时入口点/包装层之间建立桥接。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Runtime interop / 运行时互操作**:
  - **EN**: The code exposes or consumes ABI-level hooks used by generated code at runtime.
  - **CN**: 代码暴露或消费运行时由生成代码使用的 ABI 级钩子。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `VulkanRuntime.h`
- **Standard-library headers / 标准库头文件**: `<iostream>`, `<mutex>`, `<numeric>`, `<string>`, `<vector>`
