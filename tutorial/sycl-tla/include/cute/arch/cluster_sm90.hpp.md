# cluster_sm90.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/cluster_sm90.hpp`
- **EN:** Provides cluster-level coordination helpers for NVIDIA SM90 kernels.
- **CN:** 为 NVIDIA SM90 内核提供 cluster 级协作辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
#pragma once
```
- **EN:** Carries the BSD-3-Clause license banner and enables one-time inclusion with `#pragma once`.
- **CN:** 给出 BSD-3-Clause 许可证声明，并通过 `#pragma once` 启用一次性包含保护。

### Lines 33-34
```cpp
#include <cute/config.hpp>
#include <cute/numeric/numeric_types.hpp>
```
- **EN:** Imports `cute/config.hpp` (core CuTe configuration macros and portability annotations); `cute/numeric/numeric_types.hpp` (numeric helper types such as fixed-width bit wrappers).
- **CN:** 引入 `cute/config.hpp`（CuTe 核心配置宏与可移植性标注）；`cute/numeric/numeric_types.hpp`（数值辅助类型，例如固定位宽包装类型）。

### Lines 36-40
```cpp
// Config
#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 900) && \
  ((__CUDACC_VER_MAJOR__ >= 12) || ((__CUDACC_VER_MAJOR__ == 11) && (__CUDACC_VER_MINOR__ >= 8))))
#  define CUTE_ARCH_CLUSTER_SM90_ENABLED
#endif
```
- **EN:** Implements supporting control flow for the surrounding type or function.
- **CN:** 实现周边类型或函数所需的辅助控制流。

### Lines 42-44
```cpp
#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 900) && (__CUDACC_VER_MAJOR__ >= 12))
#  define CUTE_ARCH_ELECT_ONE_SM90_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (__CUDA_ARCH__, CUTE_ARCH_ELECT_ONE_SM90_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（__CUDA_ARCH__, CUTE_ARCH_ELECT_ONE_SM90_ENABLED）只启用当前目标有效的构建路径。

### Lines 46-46
```cpp
namespace cute {
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 48-55
```cpp
CUTE_DEVICE void cluster_arrive_relaxed()
{
#if defined(CUTE_ARCH_CLUSTER_SM90_ENABLED)
  asm volatile("barrier.cluster.arrive.relaxed.aligned;\n" : : );
#else
  CUTE_INVALID_CONTROL_PATH("CUTE_ARCH_CLUSTER_SM90_ENABLED is not defined");
#endif
}
```
- **EN:** Defines `volatile`, a thin wrapper around an architecture-specific inline instruction sequence.
- **CN:** 定义 `volatile`，它是对特定体系结构内联指令序列的轻量封装。

### Lines 57-64
```cpp
CUTE_DEVICE void cluster_arrive()
{
#if defined(CUTE_ARCH_CLUSTER_SM90_ENABLED)
  asm volatile("barrier.cluster.arrive.aligned;\n" : : );
#else
  CUTE_INVALID_CONTROL_PATH("CUTE_ARCH_CLUSTER_SM90_ENABLED is not defined");
#endif
}
```
- **EN:** Defines `volatile`, a thin wrapper around an architecture-specific inline instruction sequence.
- **CN:** 定义 `volatile`，它是对特定体系结构内联指令序列的轻量封装。

### Lines 66-73
```cpp
CUTE_DEVICE void cluster_wait()
{
#if defined(CUTE_ARCH_CLUSTER_SM90_ENABLED)
  asm volatile("barrier.cluster.wait.aligned;\n" : : );
#else
  CUTE_INVALID_CONTROL_PATH("CUTE_ARCH_CLUSTER_SM90_ENABLED is not defined");
#endif
}
```
- **EN:** Defines `volatile`, a thin wrapper around an architecture-specific inline instruction sequence.
- **CN:** 定义 `volatile`，它是对特定体系结构内联指令序列的轻量封装。

### Lines 75-83
```cpp
CUTE_DEVICE void cluster_sync()
{
#if defined(CUTE_ARCH_CLUSTER_SM90_ENABLED)
  cluster_arrive();
  cluster_wait();
#else
  CUTE_INVALID_CONTROL_PATH("CUTE_ARCH_CLUSTER_SM90_ENABLED is not defined");
#endif
}
```
- **EN:** Defines or forwards `cluster_arrive` as part of this header's executable interface.
- **CN:** 定义或转发 `cluster_arrive`，作为该头文件可执行接口的一部分。

### Lines 85-103
```cpp
// Returns the dim3 grid size in terms of number of clusters.
CUTE_DEVICE dim3 cluster_grid_dims()
{
#if defined(CUTE_ARCH_CLUSTER_SM90_ENABLED)
  uint32_t x, y, z;
  asm volatile("mov.u32 %0, %%nclusterid.x;\n" : "=r"(x) : );
  asm volatile("mov.u32 %0, %%nclusterid.y;\n" : "=r"(y) : );
  asm volatile("mov.u32 %0, %%nclusterid.z;\n" : "=r"(z) : );
  return {x, y, z};
#elif defined(__CUDA_ARCH__)
  // MSVC requires protecting use of gridDim with __CUDA_ARCH__.
  return gridDim;
#elif defined(_MSC_VER)
  CUTE_INVALID_CONTROL_PATH("cluster_grid_dims() can only be called on device");
  return {0, 0, 0};
#else
  return {0, 0, 0};
#endif
}
```
- **EN:** Defines `volatile`, a thin wrapper around an architecture-specific inline instruction sequence.
- **CN:** 定义 `volatile`，它是对特定体系结构内联指令序列的轻量封装。

### Lines 105-123
```cpp
// Returns the dim3 cluster rank in the grid.
CUTE_DEVICE dim3 cluster_id_in_grid()
{
#if defined(CUTE_ARCH_CLUSTER_SM90_ENABLED)
  uint32_t x, y, z;
  asm volatile("mov.u32 %0, %%clusterid.x;\n" : "=r"(x) : );
  asm volatile("mov.u32 %0, %%clusterid.y;\n" : "=r"(y) : );
  asm volatile("mov.u32 %0, %%clusterid.z;\n" : "=r"(z) : );
  return {x, y, z};
#elif defined(__CUDA_ARCH__)
  // MSVC requires protecting use of blockIdx with __CUDA_ARCH__.
  return blockIdx;
#elif defined(_MSC_VER)
  CUTE_INVALID_CONTROL_PATH("cluster_id_in_grid() can only be called on device");
  return {0, 0, 0};
#else
  return {0, 0, 0};
#endif
}
```
- **EN:** Defines `volatile`, a thin wrapper around an architecture-specific inline instruction sequence.
- **CN:** 定义 `volatile`，它是对特定体系结构内联指令序列的轻量封装。

### Lines 125-137
```cpp
// Returns the relative dim3 block rank local to the cluster.
CUTE_DEVICE dim3 block_id_in_cluster()
{
#if defined(CUTE_ARCH_CLUSTER_SM90_ENABLED)
  uint32_t x, y, z;
  asm volatile("mov.u32 %0, %%cluster_ctaid.x;\n" : "=r"(x) : );
  asm volatile("mov.u32 %0, %%cluster_ctaid.y;\n" : "=r"(y) : );
  asm volatile("mov.u32 %0, %%cluster_ctaid.z;\n" : "=r"(z) : );
  return {x, y, z};
#else
  return {0,0,0};
#endif
}
```
- **EN:** Defines `volatile`, a thin wrapper around an architecture-specific inline instruction sequence.
- **CN:** 定义 `volatile`，它是对特定体系结构内联指令序列的轻量封装。

### Lines 139-151
```cpp
// Returns the dim3 cluster shape.
CUTE_DEVICE dim3 cluster_shape()
{
#if defined(CUTE_ARCH_CLUSTER_SM90_ENABLED)
  uint32_t x, y, z;
  asm volatile("mov.u32 %0, %%cluster_nctaid.x;\n" : "=r"(x) : );
  asm volatile("mov.u32 %0, %%cluster_nctaid.y;\n" : "=r"(y) : );
  asm volatile("mov.u32 %0, %%cluster_nctaid.z;\n" : "=r"(z) : );
  return {x, y, z};
#else
  return {1,1,1};
#endif
}
```
- **EN:** Defines `volatile`, a thin wrapper around an architecture-specific inline instruction sequence.
- **CN:** 定义 `volatile`，它是对特定体系结构内联指令序列的轻量封装。

### Lines 153-163
```cpp
// Get 1D ctaid in a cluster.
CUTE_DEVICE uint32_t block_rank_in_cluster()
{
#if defined(CUTE_ARCH_CLUSTER_SM90_ENABLED)
  uint32_t rank;
  asm volatile("mov.u32 %0, %%cluster_ctarank;\n" : "=r"(rank) :);
  return rank;
#else
  return 0;
#endif
}
```
- **EN:** Defines `volatile`, a thin wrapper around an architecture-specific inline instruction sequence.
- **CN:** 定义 `volatile`，它是对特定体系结构内联指令序列的轻量封装。

### Lines 165-177
```cpp
// Set the destination block-ID in cluster for a given SMEM Address
CUTE_DEVICE uint32_t set_block_rank(uint32_t smemAddr, uint32_t rank)
{
#if defined(CUTE_ARCH_CLUSTER_SM90_ENABLED)
  uint32_t result;
  asm volatile("mapa.shared::cluster.u32  %0, %1, %2;\n"
              : "=r"(result)
              : "r"(smemAddr), "r"(rank));
  return result;
#else
  return smemAddr;
#endif
}
```
- **EN:** Defines `volatile`, a thin wrapper around an architecture-specific inline instruction sequence.
- **CN:** 定义 `volatile`，它是对特定体系结构内联指令序列的轻量封装。

### Lines 179-201
```cpp
// Elect one thread in the warp. The elected thread gets its predicate set to true, all others obtain false.
CUTE_HOST_DEVICE uint32_t elect_one_sync()
{
#if defined(CUTE_ARCH_ELECT_ONE_SM90_ENABLED)
  uint32_t pred = 0;
  uint32_t laneid = 0;
  asm volatile(
    "{\n"
    ".reg .b32 %%rx;\n"
    ".reg .pred %%px;\n"
    "     elect.sync %%rx|%%px, %2;\n"
    "@%%px mov.s32 %1, 1;\n"
    "     mov.s32 %0, %%rx;\n"
    "}\n"
    : "+r"(laneid), "+r"(pred)
    : "r"(0xFFFFFFFF));
  return pred;
#elif defined(__CUDA_ARCH__)
  return (threadIdx.x % 32) == 0;
#else
  return true;
#endif
}
```
- **EN:** Defines `volatile`, a thin wrapper around an architecture-specific inline instruction sequence.
- **CN:** 定义 `volatile`，它是对特定体系结构内联指令序列的轻量封装。

### Lines 203-206
```cpp
struct ElectOneLaneIdReturnType {
  uint32_t is_leader;
  uint32_t leader_lane_id;
};
```
- **EN:** Defines `ElectOneLaneIdReturnType` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `ElectOneLaneIdReturnType`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 208-231
```cpp
CUTE_HOST_DEVICE
ElectOneLaneIdReturnType
elect_one_leader_sync()
{
#if defined(CUTE_ARCH_ELECT_ONE_SM90_ENABLED)
  uint32_t pred = 0;
  uint32_t laneid = 0;
  asm volatile(
    "{\n"
    ".reg .b32 %%rx;\n"
    ".reg .pred %%px;\n"
    "     elect.sync %%rx|%%px, %2;\n"
    "@%%px mov.s32 %1, 1;\n"
    "     mov.s32 %0, %%rx;\n"
    "}\n"
    : "+r"(laneid), "+r"(pred)
    : "r"(0xFFFFFFFF));
  return {pred, laneid};
#elif defined(__CUDA_ARCH__)
  return {(threadIdx.x % 32) == 0, 0};
#else
  return {true, 0};
#endif
}
```
- **EN:** Defines `elect_one_leader_sync`, a thin wrapper around an architecture-specific inline instruction sequence.
- **CN:** 定义 `elect_one_leader_sync`，它是对特定体系结构内联指令序列的轻量封装。

### Lines 233-244
```cpp
// Store value to remote shared memory in the cluster
CUTE_DEVICE
void
store_shared_remote(uint32_t value, uint32_t smem_addr, uint32_t mbarrier_addr, uint32_t dst_cta_rank)
{
#if defined(CUTE_ARCH_CLUSTER_SM90_ENABLED)
  uint32_t dsmem_addr = set_block_rank(smem_addr, dst_cta_rank);
  uint32_t remote_barrier_addr = set_block_rank(mbarrier_addr, dst_cta_rank);
  asm volatile("st.async.shared::cluster.mbarrier::complete_tx::bytes.u32 [%0], %1, [%2];"
               : : "r"(dsmem_addr), "r"(value), "r"(remote_barrier_addr));
#endif
}
```
- **EN:** Defines `store_shared_remote`, a thin wrapper around an architecture-specific inline instruction sequence.
- **CN:** 定义 `store_shared_remote`，它是对特定体系结构内联指令序列的轻量封装。

### Lines 246-246
```cpp
} // end namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** Inline assembly keeps the mapping between C++ fragments and hardware registers explicit.
  **CN:** 内联汇编使 C++ 片段与硬件寄存器之间的映射保持显式。
- **EN:** Collective coordination utilities distribute work across cooperating threads or thread groups.
  **CN:** 集合协作工具把工作分发到多个协同线程或线程组上。

## Dependencies / 依赖关系

- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
- **EN:** `cute/numeric/numeric_types.hpp` supplies numeric helper types such as fixed-width bit wrappers.
  **CN:** `cute/numeric/numeric_types.hpp` 提供了数值辅助类型，例如固定位宽包装类型。
- **EN:** CUDA architecture macros gate device-only fast paths and inline instructions.
  **CN:** CUDA 架构宏用于控制仅设备端可用的快速路径与内联指令。
