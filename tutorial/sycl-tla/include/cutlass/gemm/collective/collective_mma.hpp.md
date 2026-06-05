# collective_mma.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/collective/collective_mma.hpp`
- **Purpose (EN):** Acts as the umbrella header that gathers all available `CollectiveMma` specializations for supported targets.
- **用途 (CN):** 作为总入口头文件，汇集所有已支持目标平台的 `CollectiveMma` 特化。
- **Lines / 行数:** 97

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

### Lines 33-33

```cpp
#include "cutlass/gemm/collective/collective_mma_decl.hpp"
```
**EN:** This include block imports collective_mma_decl.hpp. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 collective_mma_decl.hpp。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 36-36

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 38-79

```cpp
#include "cutlass/gemm/collective/sm70_mma_twostage.hpp"
#include "cutlass/gemm/collective/sm80_mma_multistage.hpp"
#include "cutlass/gemm/collective/sm80_mma_array_multistage.hpp"
#include "cutlass/gemm/collective/sm90_mma_multistage_gmma_ss_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm90_mma_multistage_gmma_rs_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm90_mma_tma_gmma_ss.hpp"
#include "cutlass/gemm/collective/sm90_mma_tma_gmma_rs_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm90_mma_tma_gmma_rs_warpspecialized_mixed_input.hpp" 
#include "cutlass/gemm/collective/sm90_mma_tma_gmma_ss_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm90_sparse_mma_tma_gmma_ss_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm90_sparse_mma_tma_gmma_ss_warpspecialized_fp8.hpp"
#include "cutlass/gemm/collective/sm90_mma_array_tma_gmma_ss_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm90_mma_array_tma_gmma_rs_warpspecialized_mixed_input.hpp"
#include "cutlass/gemm/collective/sm90_mma_tma_gmma_ss_warpspecialized_fp8.hpp"
#include "cutlass/gemm/collective/sm90_mma_array_tma_gmma_ss_warpspecialized_fp8.hpp"
#include "cutlass/gemm/collective/sm90_mma_tma_gmma_ss_warpspecialized_fp8_blockwise_scaling.hpp"
#include "cutlass/gemm/collective/sm90_mma_array_tma_gmma_ss_warpspecialized_fp8_blockwise_scaling.hpp"
#if !defined(__CUDACC_RTC__)
#include "cutlass/gemm/collective/sm100_mma_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm100_mma_array_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm100_mma_warpspecialized_emulated.hpp"
#include "cutlass/gemm/collective/sm100_mma_array_warpspecialized_emulated.hpp"
#include "cutlass/gemm/collective/sm100_sparse_mma_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm100_blockscaled_sparse_mma_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm100_blockscaled_mma_warpspecialized.hpp" 
#include "cutlass/gemm/collective/sm100_blockscaled_mma_array_warpspecialized.hpp" 
#include "cutlass/gemm/collective/sm100_mma_warpspecialized_blockwise_scaling.hpp"
#include "cutlass/gemm/collective/sm100_mma_array_warpspecialized_blockwise_scaling.hpp"
#include "cutlass/gemm/collective/sm100_mma_warpspecialized_mixed_input.hpp"
#include "cutlass/gemm/collective/sm100_mma_cpasync_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm100_mma_mixed_tma_cpasync_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm100_blockscaled_mma_mixed_tma_cpasync_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm103_blockscaled_mma_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm103_blockscaled_mma_array_warpspecialized.hpp"
#include "cutlass/gemm/collective/sm120_mma_tma.hpp"
#include "cutlass/gemm/collective/sm120_blockscaled_mma_tma.hpp"
#include "cutlass/gemm/collective/sm120_blockscaled_mma_array_tma.hpp"
#include "cutlass/gemm/collective/sm120_sparse_mma_tma.hpp"
#include "cutlass/gemm/collective/sm120_blockscaled_sparse_mma_tma.hpp"
#include "cutlass/gemm/collective/sm120_mma_tma_blockwise_scaling.hpp"
#include "cutlass/gemm/collective/sm120_mma_array_tma_blockwise_scaling.hpp"
#endif // !defined(__CUDACC_RTC__)
```
**EN:** This include block imports sm70_mma_twostage.hpp, sm80_mma_multistage.hpp, sm80_mma_array_multistage.hpp, sm90_mma_multistage_gmma_ss_warpspecialized.hpp, and 36 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 sm70_mma_twostage.hpp、sm80_mma_multistage.hpp、sm80_mma_array_multistage.hpp、sm90_mma_multistage_gmma_ss_warpspecialized.hpp 等 40 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 81-91

```cpp
#if defined(SYCL_INTEL_TARGET)
#include "cutlass/gemm/collective/xe_mma.hpp"
#include "cutlass/gemm/collective/xe_mma_legacy.hpp"
#include "cutlass/gemm/collective/xe_array_mma.hpp"
#include "cutlass/gemm/collective/xe_array_mma_legacy.hpp"
#include "cutlass/gemm/collective/xe_array_mma_fp8_legacy.hpp"
#include "cutlass/gemm/collective/xe_mma_mixed_input.hpp"
#include "cutlass/gemm/collective/xe_array_mma_mixed_input.hpp"
#include "cutlass/gemm/collective/xe_mma_w8a8.hpp"
#include "cutlass/gemm/collective/xe_mma_fp8_scaling.hpp"
#endif
```
**EN:** This include block imports xe_mma.hpp, xe_mma_legacy.hpp, xe_array_mma.hpp, xe_array_mma_legacy.hpp, and 5 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 xe_mma.hpp、xe_mma_legacy.hpp、xe_array_mma.hpp、xe_array_mma_legacy.hpp 等 9 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 93-95

```cpp
#if defined(CUTLASS_ENABLE_SYCL)
#include "cutlass/gemm/collective/device_agnostic_mma.hpp"
#endif
```
**EN:** This include block imports device_agnostic_mma.hpp. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 device_agnostic_mma.hpp。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 97-97

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

## Key Concepts / 关键概念

- **EN:** TMA data movement  
  **CN:** TMA 数据搬运
- **EN:** GMMA tensor-core instructions  
  **CN:** GMMA 张量核指令
- **EN:** warp-specialized scheduling  
  **CN:** warp-specialized 调度
- **EN:** multistage pipelining  
  **CN:** 多阶段流水线
- **EN:** two-stage pipelining  
  **CN:** 两阶段流水线
- **EN:** sparse MMA support  
  **CN:** 稀疏 MMA 支持
- **EN:** block-scaled numeric formats  
  **CN:** 块缩放数值格式
- **EN:** blockwise scaling  
  **CN:** 块级缩放
- **EN:** FP8 data paths  
  **CN:** FP8 数据路径
- **EN:** mixed-input operands  
  **CN:** 混合输入操作数

## Dependencies / 依赖关系

- **Direct includes / 直接包含:**
  - `cutlass/gemm/collective/collective_mma_decl.hpp`
  - `cutlass/gemm/collective/sm70_mma_twostage.hpp`
  - `cutlass/gemm/collective/sm80_mma_multistage.hpp`
  - `cutlass/gemm/collective/sm80_mma_array_multistage.hpp`
  - `cutlass/gemm/collective/sm90_mma_multistage_gmma_ss_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm90_mma_multistage_gmma_rs_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm90_mma_tma_gmma_ss.hpp`
  - `cutlass/gemm/collective/sm90_mma_tma_gmma_rs_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm90_mma_tma_gmma_rs_warpspecialized_mixed_input.hpp`
  - `cutlass/gemm/collective/sm90_mma_tma_gmma_ss_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm90_sparse_mma_tma_gmma_ss_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm90_sparse_mma_tma_gmma_ss_warpspecialized_fp8.hpp`
  - `cutlass/gemm/collective/sm90_mma_array_tma_gmma_ss_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm90_mma_array_tma_gmma_rs_warpspecialized_mixed_input.hpp`
  - `cutlass/gemm/collective/sm90_mma_tma_gmma_ss_warpspecialized_fp8.hpp`
  - `cutlass/gemm/collective/sm90_mma_array_tma_gmma_ss_warpspecialized_fp8.hpp`
  - `cutlass/gemm/collective/sm90_mma_tma_gmma_ss_warpspecialized_fp8_blockwise_scaling.hpp`
  - `cutlass/gemm/collective/sm90_mma_array_tma_gmma_ss_warpspecialized_fp8_blockwise_scaling.hpp`
  - `cutlass/gemm/collective/sm100_mma_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm100_mma_array_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm100_mma_warpspecialized_emulated.hpp`
  - `cutlass/gemm/collective/sm100_mma_array_warpspecialized_emulated.hpp`
  - `cutlass/gemm/collective/sm100_sparse_mma_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm100_blockscaled_sparse_mma_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm100_blockscaled_mma_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm100_blockscaled_mma_array_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm100_mma_warpspecialized_blockwise_scaling.hpp`
  - `cutlass/gemm/collective/sm100_mma_array_warpspecialized_blockwise_scaling.hpp`
  - `cutlass/gemm/collective/sm100_mma_warpspecialized_mixed_input.hpp`
  - `cutlass/gemm/collective/sm100_mma_cpasync_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm100_mma_mixed_tma_cpasync_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm100_blockscaled_mma_mixed_tma_cpasync_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm103_blockscaled_mma_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm103_blockscaled_mma_array_warpspecialized.hpp`
  - `cutlass/gemm/collective/sm120_mma_tma.hpp`
  - `cutlass/gemm/collective/sm120_blockscaled_mma_tma.hpp`
  - `cutlass/gemm/collective/sm120_blockscaled_mma_array_tma.hpp`
  - `cutlass/gemm/collective/sm120_sparse_mma_tma.hpp`
  - `cutlass/gemm/collective/sm120_blockscaled_sparse_mma_tma.hpp`
  - `cutlass/gemm/collective/sm120_mma_tma_blockwise_scaling.hpp`
  - `cutlass/gemm/collective/sm120_mma_array_tma_blockwise_scaling.hpp`
  - `cutlass/gemm/collective/xe_mma.hpp`
  - `cutlass/gemm/collective/xe_mma_legacy.hpp`
  - `cutlass/gemm/collective/xe_array_mma.hpp`
  - `cutlass/gemm/collective/xe_array_mma_legacy.hpp`
  - `cutlass/gemm/collective/xe_array_mma_fp8_legacy.hpp`
  - `cutlass/gemm/collective/xe_mma_mixed_input.hpp`
  - `cutlass/gemm/collective/xe_array_mma_mixed_input.hpp`
  - `cutlass/gemm/collective/xe_mma_w8a8.hpp`
  - `cutlass/gemm/collective/xe_mma_fp8_scaling.hpp`
  - `cutlass/gemm/collective/device_agnostic_mma.hpp`
- **Shared abstractions / 共享抽象:** `CollectiveMma` / `CollectiveBuilder`, dispatch-policy tags, CuTe tensor/layout utilities, and CUTLASS pipeline helpers. / `CollectiveMma` / `CollectiveBuilder`、dispatch-policy 标签、CuTe 张量/布局工具以及 CUTLASS 流水线辅助组件。