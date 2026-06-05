# kernel_hardware_info.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/kernel_hardware_info.h`

- **EN:** CUTLASS header related to kernel hardware info.

- **CN:** 该头文件属于 CUTLASS 的核心类型或工具定义。文件级摘要：CUTLASS header related to kernel hardware info.

## Line-by-Line Analysis / 逐行分析

### Lines 1-31

```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * Copyright (C) 2025 Intel Corporation, All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
```

**EN:** This block records the file copyright, SPDX identifier, and redistribution disclaimer.

**CN:** 该代码块记录了文件的版权信息、SPDX 标识以及再分发免责声明。

### Lines 32-32

```cpp
#pragma once
```

**EN:** This directive uses `#pragma once` to avoid repeated inclusion of the same header.

**CN:** 这里使用 `#pragma once` 来避免同一头文件被重复包含。

### Lines 34-34

```cpp
#include "cutlass/device_kernel.h"
```

**EN:** This block imports dependencies such as `cutlass/device_kernel.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/device_kernel.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 35-35

```cpp
#if !defined(__CUDACC_RTC__) && !defined(CUTLASS_ENABLE_SYCL)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if !defined(__CUDACC_RTC__) && !defined(CUTLASS_ENABLE_SYCL)`.

**CN:** 这个预处理代码块围绕 `#if !defined(__CUDACC_RTC__) && !defined(CUTLASS_ENABLE_SYCL)` 选择编译路径或功能开关。

### Lines 36-38

```cpp
#include "cuda_runtime.h"
#include "cutlass/cluster_launch.hpp"
#include "cutlass/trace.h"
```

**EN:** This block imports dependencies such as `cuda_runtime.h`, `cutlass/cluster_launch.hpp`, `cutlass/trace.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cuda_runtime.h`, `cutlass/cluster_launch.hpp`, `cutlass/trace.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 39-39

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 40-40

```cpp
#include <cute/int_tuple.hpp>
```

**EN:** This block imports dependencies such as `cute/int_tuple.hpp`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cute/int_tuple.hpp` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 42-42

```cpp
namespace cutlass {
```

**EN:** This block opens the namespace scope `cutlass` for the declarations that follow.

**CN:** 该代码块打开了 `cutlass` 命名空间作用域，以容纳后续声明。

### Lines 44-44

```cpp
struct KernelHardwareInfo {
```

**EN:** This block begins the definition of `KernelHardwareInfo`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `KernelHardwareInfo` 这个 `struct`，其成员会在后续代码中展开。

### Lines 45-47

```cpp
  //
  // Data members
  //
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 49-50

```cpp
  // Hardware properties
  int device_id = 0;
```

**EN:** The preceding comment documents this block. This declaration defines `device_id` and assigns it the compile-time expression `0`.

**CN:** 前面的注释说明了这个代码块。这个声明定义了 `device_id`，并把它设为编译期表达式 `0`。

### Lines 51-51

```cpp
  int sm_count  = 0;
```

**EN:** This declaration defines `sm_count` and assigns it the compile-time expression `0`.

**CN:** 这个声明定义了 `sm_count`，并把它设为编译期表达式 `0`。

### Lines 53-54

```cpp
  // Kernel properties
  int max_active_clusters = 0;              // Maximum number of clusters that could co-exist on the target device.
```

**EN:** The preceding comment documents this block. This declaration defines `max_active_clusters` and assigns it the compile-time expression `0`.

**CN:** 前面的注释说明了这个代码块。这个声明定义了 `max_active_clusters`，并把它设为编译期表达式 `0`。

### Lines 55-55

```cpp
  dim3 cluster_shape = {0,0,0};
```

**EN:** This declaration defines `cluster_shape` and assigns it the compile-time expression `{0,0,0}`.

**CN:** 这个声明定义了 `cluster_shape`，并把它设为编译期表达式 `{0,0,0}`。

### Lines 56-56

```cpp
  dim3 cluster_shape_fallback = {0,0,0};
```

**EN:** This declaration defines `cluster_shape_fallback` and assigns it the compile-time expression `{0,0,0}`.

**CN:** 这个声明定义了 `cluster_shape_fallback`，并把它设为编译期表达式 `{0,0,0}`。

### Lines 58-60

```cpp
  //
  // Methods
  //
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 62-62

```cpp
#if defined (CUTLASS_ENABLE_SYCL)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if defined (CUTLASS_ENABLE_SYCL)`.

**CN:** 这个预处理代码块围绕 `#if defined (CUTLASS_ENABLE_SYCL)` 选择编译路径或功能开关。

### Lines 63-64

```cpp
  static inline int
  query_device_multiprocessor_count(int device_id = 0) {
```

**EN:** This declaration introduces `this entity` for later use in the surrounding scope.

**CN:** 这个声明为周围作用域引入了 `this entity`，供后续代码使用。

### Lines 65-65

```cpp
    auto& dev = compat::get_device(device_id);
```

**EN:** This declaration defines `dev` and assigns it the compile-time expression `compat::get_device(device_id)`.

**CN:** 这个声明定义了 `dev`，并把它设为编译期表达式 `compat::get_device(device_id)`。

### Lines 66-66

```cpp
    int multiprocessor_count = 1;
```

**EN:** This declaration defines `multiprocessor_count` and assigns it the compile-time expression `1`.

**CN:** 这个声明定义了 `multiprocessor_count`，并把它设为编译期表达式 `1`。

### Lines 67-67

```cpp
    //TODO (Codeplay): Replace with device.get_info<sycl::ext::oneapi::info::device::num_compute_units>() once available 
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 68-68

```cpp
#if defined __SYCL_CUDA_ARCH__
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if defined __SYCL_CUDA_ARCH__`.

**CN:** 这个预处理代码块围绕 `#if defined __SYCL_CUDA_ARCH__` 选择编译路径或功能开关。

### Lines 69-69

```cpp
    multiprocessor_count = dev.get_info<sycl::info::device::max_compute_units>();
```

**EN:** This declaration defines `multiprocessor_count` and assigns it the compile-time expression `dev.get_info<sycl::info::device::max_compute_units>()`.

**CN:** 这个声明定义了 `multiprocessor_count`，并把它设为编译期表达式 `dev.get_info<sycl::info::device::max_compute_units>()`。

### Lines 70-70

```cpp
#elif defined SYCL_INTEL_TARGET
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#elif defined SYCL_INTEL_TARGET`.

**CN:** 这个预处理代码块围绕 `#elif defined SYCL_INTEL_TARGET` 选择编译路径或功能开关。

### Lines 71-71

```cpp
    multiprocessor_count = static_cast<int>(dev.get_info<sycl::ext::intel::info::device::gpu_slices>() * dev.get_info<sycl::ext::intel::info::device::gpu_subslices_per_slice>());
```

**EN:** This declaration defines `multiprocessor_count` and assigns it the compile-time expression `static_cast<int>(dev.get_info<sycl::ext::intel::info::device::gpu_slices>() * dev.get_info<sycl::ext::intel::info::device::gpu_subslices_per_slice>())`.

**CN:** 这个声明定义了 `multiprocessor_count`，并把它设为编译期表达式 `static_cast<int>(dev.get_info<sycl::ext::intel::info::device::gpu_slices>() * dev.get_info<sycl::ext::intel::info::device::gpu_subslices_per_slice>())`。

### Lines 72-72

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 73-73

```cpp
    return multiprocessor_count;
```

**EN:** This statement contributes an implementation detail to the surrounding abstraction.

**CN:** 这个语句为周围抽象补充了一个实现细节。

### Lines 75-84

```cpp
  // Query maximum number of active clusters that could co-exist on the target device
  // based on kernel properties such as cluster dims and threadblock dims
  // Return 0 for Intel Xe12 and Xe20 architectures for now
  static inline int
  query_device_max_active_clusters(
      dim3 cluster_dims,
      uint32_t threads_per_block,
      void const* kernel_ptr) {
    return 0;
  }
```

**EN:** The preceding comment documents this block. The function `query_device_max_active_clusters` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`query_device_max_active_clusters` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 87-87

```cpp
#elif !defined(__CUDACC_RTC__)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#elif !defined(__CUDACC_RTC__)`.

**CN:** 这个预处理代码块围绕 `#elif !defined(__CUDACC_RTC__)` 选择编译路径或功能开关。

### Lines 88-89

```cpp
  static inline int
  query_device_multiprocessor_count(int device_id = 0) {
```

**EN:** This declaration introduces `this entity` for later use in the surrounding scope.

**CN:** 这个声明为周围作用域引入了 `this entity`，供后续代码使用。

### Lines 90-90

```cpp
    cudaError_t result = cudaGetDevice(&device_id);
```

**EN:** This declaration defines `result` and assigns it the compile-time expression `cudaGetDevice(&device_id)`.

**CN:** 这个声明定义了 `result`，并把它设为编译期表达式 `cudaGetDevice(&device_id)`。

### Lines 91-91

```cpp
    if (result != cudaSuccess) {
```

**EN:** This declaration introduces `this entity` for later use in the surrounding scope.

**CN:** 这个声明为周围作用域引入了 `this entity`，供后续代码使用。

### Lines 92-94

```cpp
      CUTLASS_TRACE_HOST(
        "  cudaGetDevice() returned error "
        << cudaGetErrorString(result));
```

**EN:** The function `CUTLASS_TRACE_HOST` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `CUTLASS_TRACE_HOST` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 95-95

```cpp
      return 0;
```

**EN:** This statement contributes an implementation detail to the surrounding abstraction.

**CN:** 这个语句为周围抽象补充了一个实现细节。

### Lines 97-97

```cpp
    int multiprocessor_count;
```

**EN:** This statement contributes an implementation detail to the surrounding abstraction.

**CN:** 这个语句为周围抽象补充了一个实现细节。

### Lines 98-99

```cpp
    result = cudaDeviceGetAttribute(&multiprocessor_count,
      cudaDevAttrMultiProcessorCount, device_id);
```

**EN:** This declaration defines `result` and assigns it the compile-time expression `cudaDeviceGetAttribute(&multiprocessor_count, cudaDevAttrMultiProcessorCount, device_id)`.

**CN:** 这个声明定义了 `result`，并把它设为编译期表达式 `cudaDeviceGetAttribute(&multiprocessor_count, cudaDevAttrMultiProcessorCount, device_id)`。

### Lines 100-100

```cpp
    if (result != cudaSuccess) {
```

**EN:** This declaration introduces `this entity` for later use in the surrounding scope.

**CN:** 这个声明为周围作用域引入了 `this entity`，供后续代码使用。

### Lines 101-103

```cpp
      CUTLASS_TRACE_HOST(
        "  cudaDeviceGetAttribute() returned error "
        << cudaGetErrorString(result));
```

**EN:** The function `CUTLASS_TRACE_HOST` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `CUTLASS_TRACE_HOST` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 104-104

```cpp
      return 0;
```

**EN:** This statement contributes an implementation detail to the surrounding abstraction.

**CN:** 这个语句为周围抽象补充了一个实现细节。

### Lines 106-106

```cpp
    return multiprocessor_count;
```

**EN:** This statement contributes an implementation detail to the surrounding abstraction.

**CN:** 这个语句为周围抽象补充了一个实现细节。

### Lines 109-135

```cpp
  // Query maximum number of active clusters that could co-exist on the target device
  // based on kernel properties such as cluster dims and threadblock dims
  static inline int
  query_device_max_active_clusters(
      dim3 cluster_dims,
      uint32_t threads_per_block,
      void const* kernel_ptr) {
    int max_active_clusters = 0;
#if defined(CUTLASS_SM90_CLUSTER_LAUNCH_ENABLED)
    ClusterLauncher::LaunchConfig cluster_launch_config = ClusterLauncher::make_cluster_launch_config(
                                                            cluster_dims /* minimum grid dim */, cluster_dims, {threads_per_block, 1, 1});
    // Given the kernel function and launch configuration, return the maximum number of clusters that could co-exist on the target device.
    cudaError_t result = cudaOccupancyMaxActiveClusters(&max_active_clusters, kernel_ptr, &cluster_launch_config.launch_config);
    if (result != cudaSuccess) {
      CUTLASS_TRACE_HOST(
        "  cudaGetDevice() returned error "
        << cudaGetErrorString(result));
      return 0;
    }
    CUTLASS_TRACE_HOST("cudaOccupancyMaxActiveClusters: maximum number of clusters that could co-exist on the target device = "
        << max_active_clusters << "\n");
    return max_active_clusters;
#else
    CUTLASS_TRACE_HOST("ClusterLauncher: CUTLASS_SM90_CLUSTER_LAUNCH_ENABLED not defined! Aborting cluster occupancy query.");
    return max_active_clusters;
#endif
  }
```

**EN:** The preceding comment documents this block. The function `max_active_clusters` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`max_active_clusters` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 137-147

```cpp
  // Simpler version of the above query function that fetches relevant information from the Kernel
  template <typename Kernel>
  static inline int
  query_device_max_active_clusters() {
    dim3 cluster_dims(cute::size<0>(typename Kernel::ClusterShape{}),
                      cute::size<1>(typename Kernel::ClusterShape{}),
                      cute::size<2>(typename Kernel::ClusterShape{}));
    uint32_t threads_per_block = Kernel::MaxThreadsPerBlock;
    void const* kernel_ptr = (void*)(device_kernel<Kernel>);
    return query_device_max_active_clusters(cluster_dims, threads_per_block, kernel_ptr);
  }
```

**EN:** The preceding comment documents this block. The function `threads_per_block` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`threads_per_block` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 149-151

```cpp
  template <typename Kernel>
  static inline KernelHardwareInfo
  make_kernel_hardware_info(int const device_id = 0, int sm_count = 0, int max_active_clusters = 0) {
```

**EN:** This declaration introduces `this entity` for later use in the surrounding scope.

**CN:** 这个声明为周围作用域引入了 `this entity`，供后续代码使用。

### Lines 152-152

```cpp
    if (sm_count == 0) {
```

**EN:** This declaration introduces `this entity` for later use in the surrounding scope.

**CN:** 这个声明为周围作用域引入了 `this entity`，供后续代码使用。

### Lines 153-153

```cpp
      sm_count = query_device_multiprocessor_count(device_id);
```

**EN:** This declaration defines `sm_count` and assigns it the compile-time expression `query_device_multiprocessor_count(device_id)`.

**CN:** 这个声明定义了 `sm_count`，并把它设为编译期表达式 `query_device_multiprocessor_count(device_id)`。

### Lines 155-155

```cpp
    if (max_active_clusters == 0) {
```

**EN:** This declaration introduces `this entity` for later use in the surrounding scope.

**CN:** 这个声明为周围作用域引入了 `this entity`，供后续代码使用。

### Lines 156-156

```cpp
      max_active_clusters = query_device_max_active_clusters<Kernel>();
```

**EN:** This declaration defines `max_active_clusters` and assigns it the compile-time expression `query_device_max_active_clusters<Kernel>()`.

**CN:** 这个声明定义了 `max_active_clusters`，并把它设为编译期表达式 `query_device_max_active_clusters<Kernel>()`。

### Lines 158-158

```cpp
    return {device_id, sm_count, max_active_clusters};
```

**EN:** This statement contributes an implementation detail to the surrounding abstraction.

**CN:** 这个语句为周围抽象补充了一个实现细节。

### Lines 160-160

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

## Key Concepts / 关键概念

- **EN:** Heavy use of templates enables compile-time specialization and zero-overhead abstractions.
  **CN:** 大量使用模板，使该文件能够进行编译期特化并保持零额外开销的抽象。

- **EN:** Conditional compilation keeps the header portable across host, device, CUDA, and SYCL builds.
  **CN:** 条件编译使该头文件能够在主机端、设备端、CUDA 与 SYCL 构建之间保持可移植性。

## Dependencies / 依赖关系

- **EN:** Direct includes: `cutlass/device_kernel.h`, `cuda_runtime.h`, `cutlass/cluster_launch.hpp`, `cutlass/trace.h`, `cute/int_tuple.hpp`.
  **CN:** 直接包含：`cutlass/device_kernel.h`, `cuda_runtime.h`, `cutlass/cluster_launch.hpp`, `cutlass/trace.h`, `cute/int_tuple.hpp`。

- **EN:** Primary namespaces: `cutlass`.
  **CN:** 主要命名空间：`cutlass`。

- **EN:** Important macros or compile flags: `CUTLASS_ENABLE_SYCL`, `CUTLASS_SM90_CLUSTER_LAUNCH_ENABLED`, `CUTLASS_TRACE_HOST`.
  **CN:** 重要宏或编译开关：`CUTLASS_ENABLE_SYCL`, `CUTLASS_SM90_CLUSTER_LAUNCH_ENABLED`, `CUTLASS_TRACE_HOST`。
