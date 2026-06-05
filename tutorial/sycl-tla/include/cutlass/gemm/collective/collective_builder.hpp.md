# collective_builder.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/collective/collective_builder.hpp`
- **Purpose (EN):** Aggregates collective builder declarations and architecture-specific builder `.inl` implementations.
- **用途 (CN):** 汇总 collective builder 的声明以及各架构对应的 `.inl` builder 实现。
- **Lines / 行数:** 69

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
**EN:** This opening block combines the license banner with `#pragma once`, documenting legal terms and making the header safe to include multiple times.
**CN:** 这个开头块同时包含许可证说明和 `#pragma once`：前者给出法律信息，后者保证头文件可被重复包含而不重复定义。

### Lines 33-35

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
#include "cutlass/gemm/collective/collective_mma_decl.hpp"
#include "cutlass/gemm/collective/collective_mma.hpp"
```
**EN:** This include block imports collective_mma_decl.hpp, collective_mma.hpp. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 collective_mma_decl.hpp、collective_mma.hpp。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 37-37

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 39-60

```cpp
#include "cutlass/gemm/collective/collective_builder_decl.hpp"
#include "cutlass/gemm/collective/builders/sm90_gmma_builder.inl"
#include "cutlass/gemm/collective/builders/sm90_sparse_gmma_builder.inl"
#if !defined(__CUDACC_RTC__) 
#include "cutlass/gemm/collective/builders/sm100_umma_builder.inl"              
#include "cutlass/gemm/collective/builders/sm100_9xBF16_umma_builder.inl"       
#include "cutlass/gemm/collective/builders/sm100_sparse_umma_builder.inl"
#include "cutlass/gemm/collective/builders/sm100_blockscaled_umma_builder.inl"
#include "cutlass/gemm/collective/builders/sm100_blockwise_umma_builder.inl"
#include "cutlass/gemm/collective/builders/sm100_blockscaled_sparse_umma_builder.inl"
#include "cutlass/gemm/collective/builders/sm100_simt_builder.inl"
#include "cutlass/gemm/collective/builders/sm100_mixed_input_umma_builder.inl"       
#include "cutlass/gemm/collective/builders/sm100_cpasync_umma_builder.inl"
#include "cutlass/gemm/collective/builders/sm100_mixed_tma_cpasync_umma_builder.inl"
#include "cutlass/gemm/collective/builders/sm100_blockscaled_mixed_tma_cpasync_umma_builder.inl"
#include "cutlass/gemm/collective/builders/sm103_blockscaled_umma_builder.inl"
#include "cutlass/gemm/collective/builders/sm120_mma_builder.inl"
#include "cutlass/gemm/collective/builders/sm120_blockscaled_mma_builder.inl"
#include "cutlass/gemm/collective/builders/sm120_sparse_mma_builder.inl"
#include "cutlass/gemm/collective/builders/sm120_blockscaled_sparse_mma_builder.inl"
#include "cutlass/gemm/collective/builders/sm120_blockwise_mma_builder.inl"
#endif
```
**EN:** This include block imports collective_builder_decl.hpp, sm90_gmma_builder.inl, sm90_sparse_gmma_builder.inl, sm100_umma_builder.inl, and 16 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 collective_builder_decl.hpp、sm90_gmma_builder.inl、sm90_sparse_gmma_builder.inl、sm100_umma_builder.inl 等 20 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 62-64

```cpp
#if defined(SYCL_INTEL_TARGET)
#include "cutlass/gemm/collective/builders/xe_mma_builder.inl"
#endif
```
**EN:** This include block imports xe_mma_builder.inl. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 xe_mma_builder.inl。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 66-69

```cpp
#if defined(CUTLASS_ENABLE_SYCL)
#include "cutlass/gemm/collective/builders/device_agnostic_mma_builder.inl"
#endif
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This include block imports device_agnostic_mma_builder.inl. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 device_agnostic_mma_builder.inl。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

## Key Concepts / 关键概念

- **EN:** TMA data movement  
  **CN:** TMA 数据搬运
- **EN:** GMMA tensor-core instructions  
  **CN:** GMMA 张量核指令
- **EN:** UMMA tensor-core instructions  
  **CN:** UMMA 张量核指令
- **EN:** sparse MMA support  
  **CN:** 稀疏 MMA 支持
- **EN:** block-scaled numeric formats  
  **CN:** 块缩放数值格式
- **EN:** blockwise scaling  
  **CN:** 块级缩放
- **EN:** mixed-input operands  
  **CN:** 混合输入操作数
- **EN:** cp.async copy stages  
  **CN:** cp.async 拷贝阶段
- **EN:** Intel Xe subgroup execution  
  **CN:** Intel Xe 子组执行
- **EN:** SYCL portability hooks  
  **CN:** SYCL 可移植性钩子

## Dependencies / 依赖关系

- **Direct includes / 直接包含:**
  - `cutlass/gemm/collective/collective_mma_decl.hpp`
  - `cutlass/gemm/collective/collective_mma.hpp`
  - `cutlass/gemm/collective/collective_builder_decl.hpp`
  - `cutlass/gemm/collective/builders/sm90_gmma_builder.inl`
  - `cutlass/gemm/collective/builders/sm90_sparse_gmma_builder.inl`
  - `cutlass/gemm/collective/builders/sm100_umma_builder.inl`
  - `cutlass/gemm/collective/builders/sm100_9xBF16_umma_builder.inl`
  - `cutlass/gemm/collective/builders/sm100_sparse_umma_builder.inl`
  - `cutlass/gemm/collective/builders/sm100_blockscaled_umma_builder.inl`
  - `cutlass/gemm/collective/builders/sm100_blockwise_umma_builder.inl`
  - `cutlass/gemm/collective/builders/sm100_blockscaled_sparse_umma_builder.inl`
  - `cutlass/gemm/collective/builders/sm100_simt_builder.inl`
  - `cutlass/gemm/collective/builders/sm100_mixed_input_umma_builder.inl`
  - `cutlass/gemm/collective/builders/sm100_cpasync_umma_builder.inl`
  - `cutlass/gemm/collective/builders/sm100_mixed_tma_cpasync_umma_builder.inl`
  - `cutlass/gemm/collective/builders/sm100_blockscaled_mixed_tma_cpasync_umma_builder.inl`
  - `cutlass/gemm/collective/builders/sm103_blockscaled_umma_builder.inl`
  - `cutlass/gemm/collective/builders/sm120_mma_builder.inl`
  - `cutlass/gemm/collective/builders/sm120_blockscaled_mma_builder.inl`
  - `cutlass/gemm/collective/builders/sm120_sparse_mma_builder.inl`
  - `cutlass/gemm/collective/builders/sm120_blockscaled_sparse_mma_builder.inl`
  - `cutlass/gemm/collective/builders/sm120_blockwise_mma_builder.inl`
  - `cutlass/gemm/collective/builders/xe_mma_builder.inl`
  - `cutlass/gemm/collective/builders/device_agnostic_mma_builder.inl`
- **Shared abstractions / 共享抽象:** `CollectiveMma` / `CollectiveBuilder`, dispatch-policy tags, CuTe tensor/layout utilities, and CUTLASS pipeline helpers. / `CollectiveMma` / `CollectiveBuilder`、dispatch-policy 标签、CuTe 张量/布局工具以及 CUTLASS 流水线辅助组件。