# helper.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `examples/common/helper.h`
- **Purpose / 目的:** Defines reusable declarations and helper logic for the repository's shared helper utilities. / 为仓库中的共享辅助工具提供可复用的声明与辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14
````cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
````
**EN:** This opening block carries the license banner and file-level description, framing the shared helper utilities example before the executable code begins. It corresponds to block 1 of 12 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代共享辅助工具示例的背景。 它对应本文件顺序中的第 1/12 个代码块。

### Lines 15-28
````cpp
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
````
**EN:** This block continues the file's shared helper utilities setup or compute path, with `Neither`, `the`, `name`, `of` showing the main symbols being prepared or consumed here. It corresponds to block 2 of 12 in the file order.
**CN:** 这一段继续推进本文件的共享辅助工具初始化或计算流程，其中 `Neither`、`the`、`name`、`of` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 2/12 个代码块。

### Lines 29-31
````cpp
 *
 **************************************************************************************************/
#pragma once
````
**EN:** This block continues the file's shared helper utilities setup or compute path, with `pragma`, `once` showing the main symbols being prepared or consumed here. It corresponds to block 3 of 12 in the file order.
**CN:** 这一段继续推进本文件的共享辅助工具初始化或计算流程，其中 `pragma`、`once` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 3/12 个代码块。

### Lines 33-38
````cpp
#if defined(CUTLASS_ENABLE_SYCL)
#include "cutlass/util/sycl_timer.hpp"
#else
#include <cuda_runtime.h>
#endif
#include <iostream>
````
**EN:** This block pulls in dependencies required by the file, especially `SYCL`, `sycl`, `cuda`, so the later shared helper utilities code can use the needed APIs and data structures. It corresponds to block 4 of 12 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `SYCL`、`sycl`、`cuda`，使后续共享辅助工具代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 4/12 个代码块。

### Lines 40-51
````cpp
/**
 * Panic wrapper for unwinding CUTLASS errors
 */
#define CUTLASS_CHECK(status)                                                                    \
  {                                                                                              \
    cutlass::Status error = status;                                                              \
    if (error != cutlass::Status::kSuccess) {                                                    \
      std::cerr << "Got cutlass error: " << cutlassGetStatusString(error) << " at: " << __LINE__ \
                << std::endl;                                                                    \
      exit(EXIT_FAILURE);                                                                        \
    }                                                                                            \
  }
````
**EN:** This opening block carries the license banner and file-level description, framing the shared helper utilities example before the executable code begins. It corresponds to block 5 of 12 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代共享辅助工具示例的背景。 它对应本文件顺序中的第 5/12 个代码块。

### Lines 54-65
````cpp
/**
 * Panic wrapper for unwinding CUDA runtime errors
 */
#define CUDA_CHECK(status)                                              \
  {                                                                     \
    cudaError_t error = status;                                         \
    if (error != cudaSuccess) {                                         \
      std::cerr << "Got bad cuda status: " << cudaGetErrorString(error) \
                << " at line: " << __LINE__ << std::endl;               \
      exit(EXIT_FAILURE);                                               \
    }                                                                   \
  }
````
**EN:** This opening block carries the license banner and file-level description, framing the shared helper utilities example before the executable code begins. It corresponds to block 6 of 12 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代共享辅助工具示例的背景。 它对应本文件顺序中的第 6/12 个代码块。

### Lines 68-79
````cpp
/**
 * GPU timer for recording the elapsed time across kernel(s) launched in GPU stream
 */
struct GpuTimer {
#if defined(CUTLASS_ENABLE_SYCL)
    using cudaStream_t = int;
    SYCLTimer syclTimer;
#else
    cudaEvent_t _start;
    cudaEvent_t _stop;
#endif
    cudaStream_t _stream_id;
````
**EN:** This opening block carries the license banner and file-level description, framing the shared helper utilities example before the executable code begins. It corresponds to block 7 of 12 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代共享辅助工具示例的背景。 它对应本文件顺序中的第 7/12 个代码块。

### Lines 81-88
````cpp
    /// Constructor
    GpuTimer() : _stream_id(0)
    {
#if !defined(CUTLASS_ENABLE_SYCL)
        CUDA_CHECK(cudaEventCreate(&_start));
        CUDA_CHECK(cudaEventCreate(&_stop));
#endif
    }
````
**EN:** This block continues the file's shared helper utilities setup or compute path, with `SYCL`, `CUDA`, `cuda` showing the main symbols being prepared or consumed here. It corresponds to block 8 of 12 in the file order.
**CN:** 这一段继续推进本文件的共享辅助工具初始化或计算流程，其中 `SYCL`、`CUDA`、`cuda` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 8/12 个代码块。

### Lines 90-97
````cpp
    /// Destructor
    ~GpuTimer()
    {
#if !defined(CUTLASS_ENABLE_SYCL)
        CUDA_CHECK(cudaEventDestroy(_start));
        CUDA_CHECK(cudaEventDestroy(_stop));
#endif
    }
````
**EN:** This block continues the file's shared helper utilities setup or compute path, with `SYCL`, `CUDA`, `cuda` showing the main symbols being prepared or consumed here. It corresponds to block 9 of 12 in the file order.
**CN:** 这一段继续推进本文件的共享辅助工具初始化或计算流程，其中 `SYCL`、`CUDA`、`cuda` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 9/12 个代码块。

### Lines 99-108
````cpp
    /// Start the timer for a given stream (defaults to the default stream)
    void start(cudaStream_t stream_id = 0)
    {
        _stream_id = stream_id;
#if defined(CUTLASS_ENABLE_SYCL)
        syclTimer.start();
#else
        CUDA_CHECK(cudaEventRecord(_start, _stream_id));
#endif
    }
````
**EN:** This block continues the file's shared helper utilities setup or compute path, with `SYCL`, `sycl`, `CUDA`, `cuda` showing the main symbols being prepared or consumed here. It corresponds to block 10 of 12 in the file order.
**CN:** 这一段继续推进本文件的共享辅助工具初始化或计算流程，其中 `SYCL`、`sycl`、`CUDA`、`cuda` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 10/12 个代码块。

### Lines 110-118
````cpp
    /// Stop the timer
    void stop()
    {
#if defined(CUTLASS_ENABLE_SYCL)
        syclTimer.stop();
#else
        CUDA_CHECK(cudaEventRecord(_stop, _stream_id));
#endif
    }
````
**EN:** This block continues the file's shared helper utilities setup or compute path, with `SYCL`, `sycl`, `CUDA`, `cuda` showing the main symbols being prepared or consumed here. It corresponds to block 11 of 12 in the file order.
**CN:** 这一段继续推进本文件的共享辅助工具初始化或计算流程，其中 `SYCL`、`sycl`、`CUDA`、`cuda` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 11/12 个代码块。

### Lines 120-132
````cpp
    /// Return the elapsed time (in milliseconds)
    float elapsed_millis()
    {
#if defined(CUTLASS_ENABLE_SYCL)
        return syclTimer.milliseconds();
#else
        float elapsed = 0.0;
        CUDA_CHECK(cudaEventSynchronize(_stop));
        CUDA_CHECK(cudaEventElapsedTime(&elapsed, _start, _stop));
        return elapsed;
#endif
    }
};
````
**EN:** This block finalizes a local computation or status path. The use of `SYCL`, `sycl`, `CUDA`, `cuda` helps conclude the current stage cleanly before the next block. It corresponds to block 12 of 12 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `SYCL`、`sycl`、`CUDA`、`cuda`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 12/12 个代码块。

## Key Concepts / 关键概念
- **EN:** SYCL objects and queues act as the execution interface for heterogeneous kernels.
  **CN:** SYCL 对象与队列构成异构内核的执行接口。
- **EN:** Header-only abstractions centralize reusable declarations and launch helpers.
  **CN:** 头文件抽象把可复用声明与启动辅助逻辑集中起来。
- **EN:** The file mixes compile-time configuration with runtime problem sizes or arguments.
  **CN:** 该文件把编译期配置与运行期问题规模或参数结合起来。
- **EN:** Validation, benchmarking, or reporting code is interleaved with kernel setup.
  **CN:** 校验、基准测试或结果报告代码与内核配置过程交织在一起。

## Dependencies / 依赖关系
- **Direct dependencies / 直接依赖:** `cutlass/util/sycl_timer.hpp`, `cuda_runtime.h`, `iostream`
- **Runtime expectations / 运行时依赖:** SYCL runtime / SYCL 运行时; CUDA runtime / CUDA 运行时; CUTLASS/CUTE templates / CUTLASS/CUTE 模板
- **Build/context note / 构建说明:** The file relies on surrounding repository infrastructure, compile flags, and example utilities. / 该文件依赖仓库周边基础设施、编译选项以及示例辅助代码。
