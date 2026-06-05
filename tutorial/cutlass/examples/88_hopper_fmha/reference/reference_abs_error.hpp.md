# reference_abs_error.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/88_hopper_fmha/reference/reference_abs_error.hpp`  
**Purpose / 用途**: Computes absolute-difference statistics between optimized outputs and reference outputs, including max error, mean error, and optional per-element printing. / 计算优化输出与参考输出之间的绝对误差统计，包括最大误差、平均误差，以及可选的逐元素打印。

---

## Line-by-Line Analysis / 逐行分析

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

**EN**: This comment block frames the file: it usually states the target architecture, problem family, usage pattern, or design motivation before the code starts. That context matters because later template choices are easier to understand once the intended hardware path and workload are known. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一段注释用于给文件定调：通常先说明目标架构、问题类型、使用方式或设计动机，再进入具体代码。先理解这些背景，再看后面的模板选择与硬件路径会更清晰。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
#pragma once

#include <cmath>
#include "cutlass/util/device_memory.h"

template<typename Element>
__global__ void reference_abs_diff_kernel(
    Element* data, Element* data_ref, size_t count,
    double* max_diff, double* sum_diff,
    bool print_diff
) {
    double thread_max_diff = 0;
    double thread_sum_diff = 0;

    __shared__ double block_max_diff;
    __shared__ double block_sum_diff;

    for (size_t i = threadIdx.x + blockIdx.x * blockDim.x; i < count; i += blockDim.x * gridDim.x) {
      double diff = fabs(data[i] - data_ref[i]);
      if (print_diff) if (diff != diff || diff > 0.01f) printf("difference at %lld: %f ... %f vs %f\n", static_cast<long long int>(i), diff, (double)data[i], (double)data_ref[i]);
      thread_max_diff = fmax(diff, thread_max_diff);
      thread_sum_diff += diff;
    }

    for (int i = 0; i < blockDim.x; i++) {
      if (i == threadIdx.x) {
        if (i == 0) {
          block_max_diff = thread_max_diff;
          block_sum_diff = thread_sum_diff;
        } else {
          block_max_diff = fmax(block_max_diff, thread_max_diff);
          block_sum_diff += thread_sum_diff;
        }
      }
      __syncthreads();
   }

   if (threadIdx.x == 0) {
     atomicAdd(sum_diff, block_sum_diff);
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
     for (;;) {
       unsigned long long prev = *reinterpret_cast<unsigned long long*>(max_diff);
       double prev_diff = reinterpret_cast<double const&>(prev);
       double new_max_diff = fmax(block_max_diff, prev_diff);
       unsigned long long found = atomicCAS(reinterpret_cast<unsigned long long*>(max_diff), prev, reinterpret_cast<unsigned long long const&>(new_max_diff));
       if (found == prev) break;
    }
   }
}

template<typename Element>
void reference_abs_diff(
    cutlass::DeviceAllocation<Element> const& data,
    cutlass::DeviceAllocation<Element> const& data_ref,
    double& max_diff, double& mean_diff
) {
  static bool kPrintDiff = getenv("REF_PRINT_DIFF") && atoi(getenv("REF_PRINT_DIFF")) == 1;

  cutlass::DeviceAllocation<double> result;
  result.reset(2);
  assert(data.size() == data_ref.size());

  cudaError_t err = cudaMemset(result.get(), 0, result.size() * sizeof(double));
  if (err != cudaSuccess) {
    std::cerr << "Memset failed. Last CUDA error: "
              << cudaGetErrorString(err) << std::endl;
    max_diff = mean_diff = 1e20;
    return;
  }

  dim3 block(256, 1, 1);
  dim3 grid(1024, 1, 1);
  reference_abs_diff_kernel<<<block, grid>>>(
      data.get(), data_ref.get(), data.size(),
      result.get(), result.get() + 1, kPrintDiff);
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  err = cudaDeviceSynchronize();
  if (err != cudaSuccess) {
    std::cerr << "Difference kernel failed. Last CUDA error: "
              << cudaGetErrorString(err) << std::endl;
    max_diff = mean_diff = 1e20;
    return;
  }

  double result_host[2];
  err = cudaMemcpy(result_host, result.get(), result.size() * sizeof(double), cudaMemcpyDefault);
  if (err != cudaSuccess) {
    std::cerr << "Copy failed. Last CUDA error: "
              << cudaGetErrorString(err) << std::endl;
    max_diff = mean_diff = 1e20;
    return;
  }
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  max_diff = result_host[0];
  mean_diff = result_host[1] / static_cast<double>(data.size());
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层
- Reference-based numerical validation / 基于参考实现的数值验证

## Dependencies / 依赖项

- `cmath` — included dependency used by this example / 此示例使用的包含依赖
- `cutlass/util/device_memory.h` — CUTLASS utility support code / CUTLASS 工具支持代码
