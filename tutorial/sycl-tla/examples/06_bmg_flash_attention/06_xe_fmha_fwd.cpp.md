# 06_xe_fmha_fwd.cpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `examples/06_bmg_flash_attention/06_xe_fmha_fwd.cpp`
- **Purpose / 目的:** Demonstrates and validates the repository's flashattention / attention implementation. / 演示并验证仓库中的FlashAttention / 注意力实现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14
````cpp
/***************************************************************************************************
 * Copyright (C) 2024 - 2025 Codeplay Software Ltd. All rights reserved.
 * Copyright (C) 2025 - 2026 Intel Corporation, All rights reserved.
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
````
**EN:** This opening block carries the license banner and file-level description, framing the flashattention / attention example before the executable code begins. It corresponds to block 1 of 31 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代FlashAttention / 注意力示例的背景。 它对应本文件顺序中的第 1/31 个代码块。

### Lines 15-28
````cpp
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
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Neither`, `the`, `name`, `of` showing the main symbols being prepared or consumed here. It corresponds to block 2 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Neither`、`the`、`name`、`of` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 2/31 个代码块。

### Lines 29-33
````cpp
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
/*! \file
    \brief Flash Attention V2 Prefill for Intel BMG
````
**EN:** This opening block carries the license banner and file-level description, framing the flashattention / attention example before the executable code begins. It corresponds to block 3 of 31 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代FlashAttention / 注意力示例的背景。 它对应本文件顺序中的第 3/31 个代码块。

### Lines 35-37
````cpp
    This example constructs and executes a Flash Attention Prefill kernel on Intel BMG. The
    definition of the GEMM, options etc for this example are defined in the associated
    bmg_flash_attn_runner.hpp header file.
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `cute`, `runner` showing the main symbols being prepared or consumed here. It corresponds to block 4 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `cute`、`runner` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 4/31 个代码块。

### Lines 39-39
````cpp
    See https://arxiv.org/pdf/2307.08691 for details of Flash Attention V2 algorithm
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `See`, `https`, `arxiv`, `org` showing the main symbols being prepared or consumed here. It corresponds to block 5 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `See`、`https`、`arxiv`、`org` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 5/31 个代码块。

### Lines 41-43
````cpp
    To run this example:
      $ ./examples/sycl/06_bmg_flash_attention/06_xe_fmha_fwd --seq_len_qo=512
        --seq_len_kv=512 --head_size_vo=128 --head_size_qk=128
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 6 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 6/31 个代码块。

### Lines 45-45
````cpp
    To build & run this example (from your build dir):
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `To`, `build`, `run`, `this` showing the main symbols being prepared or consumed here. It corresponds to block 7 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `To`、`build`、`run`、`this` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 7/31 个代码块。

### Lines 47-48
````cpp
      $ ninja 06_xe_fmha_fwd
      $ ./examples/sycl/06_bmg_flash_attention/06_xe_fmha_fwd
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 8 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 8/31 个代码块。

### Lines 50-51
````cpp
    Call with `--help` for information about available options
*/
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Call`, `with`, `help`, `information` showing the main symbols being prepared or consumed here. It corresponds to block 9 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Call`、`with`、`help`、`information` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 9/31 个代码块。

### Lines 53-53
````cpp
#include "xe_fmha_fwd_runner.hpp"
````
**EN:** This block pulls in dependencies required by the file, especially `runner`, so the later flashattention / attention code can use the needed APIs and data structures. It corresponds to block 10 of 31 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `runner`，使后续FlashAttention / 注意力代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 10/31 个代码块。

### Lines 55-58
````cpp
int main(int argc, const char **argv) {
  //
  // Parse options
  //
````
**EN:** This block introduces executable logic through a function or method. Here, `main`, `argc`, `char`, `argv` drive a concrete step in the file's flashattention / attention flow. It corresponds to block 11 of 31 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `main`、`argc`、`char`、`argv` 推动了本文件FlashAttention / 注意力流程中的一个具体步骤。 它对应本文件顺序中的第 11/31 个代码块。

### Lines 60-60
````cpp
  Options options;
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Options`, `options` showing the main symbols being prepared or consumed here. It corresponds to block 12 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Options`、`options` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 12/31 个代码块。

### Lines 62-62
````cpp
  options.parse(argc, argv);
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `options`, `parse`, `argc`, `argv` showing the main symbols being prepared or consumed here. It corresponds to block 13 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `options`、`parse`、`argc`、`argv` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 13/31 个代码块。

### Lines 64-67
````cpp
  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }
````
**EN:** This block applies conditional control flow. It uses `options`, `help`, `print_usage` to select a path, validate assumptions, or handle special cases in the flashattention / attention implementation. It corresponds to block 14 of 31 in the file order.
**CN:** 这一段实现条件控制流。它借助 `options`、`help`、`print_usage` 在FlashAttention / 注意力实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 14/31 个代码块。

### Lines 69-72
````cpp
  if (options.error) {
    std::cerr << "Aborting execution." << std::endl;
    return -1;
  }
````
**EN:** This block applies conditional control flow. It uses `options`, `error`, `Aborting`, `execution` to select a path, validate assumptions, or handle special cases in the flashattention / attention implementation. It corresponds to block 15 of 31 in the file order.
**CN:** 这一段实现条件控制流。它借助 `options`、`error`、`Aborting`、`execution` 在FlashAttention / 注意力实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 15/31 个代码块。

### Lines 74-74
````cpp
  // Define the work-group tile shape depending on the head-size of the second matmul
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Define`, `the`, `work`, `group` showing the main symbols being prepared or consumed here. It corresponds to block 16 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Define`、`the`、`work`、`group` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 16/31 个代码块。

### Lines 76-82
````cpp
#ifdef PREFILL
#if HEAD_DIM == 16
  /* Tiny config for testing */
  using ShapeQK = Shape<_1, _16, _16>;       // (q,k,d)
  using ShapePV = Shape<_1, _16, _16>;       // (q,v,k)
  using ShapeOut = Shape<_1, _16>;           // (q,v)
  using SubgroupLayoutQK = Layout<Shape<_1, _1, _1>>;
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Shape`, `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 17 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Shape`、`Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 17/31 个代码块。

### Lines 84-88
````cpp
#elif HEAD_DIM == 64
  using ShapeQK = Shape<_128, _64, _32>;
  using ShapePV = Shape<_128, _32, _64>;
  using ShapeOut = Shape<_128, _64>;
  using SubgroupLayoutQK = Layout<Shape<_8, _1, _1>>;
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Shape`, `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 18 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Shape`、`Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 18/31 个代码块。

### Lines 90-94
````cpp
#elif HEAD_DIM == 96
  using ShapeQK = Shape<_128, _64, _32>;
  using ShapePV = Shape<_128, _32, _64>;
  using ShapeOut = Shape<_128, _96>;
  using SubgroupLayoutQK = Layout<Shape<_8, _1, _1>>;
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Shape`, `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 19 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Shape`、`Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 19/31 个代码块。

### Lines 96-100
````cpp
#elif HEAD_DIM == 128
  using ShapeQK = Shape<_256, _32, _32>;
  using ShapePV = Shape<_256, _32, _32>;
  using ShapeOut = Shape<_256, _128>;
  using SubgroupLayoutQK = Layout<Shape<_16, _1, _1>>;
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Shape`, `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 20 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Shape`、`Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 20/31 个代码块。

### Lines 102-106
````cpp
#elif HEAD_DIM == 192
  using ShapeQK = Shape<_256, _64, _32>;
  using ShapePV = Shape<_256, _32, _64>;
  using ShapeOut = Shape<_256, _192>;
  using SubgroupLayoutQK = Layout<Shape<_32, _1, _1>>;
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Shape`, `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 21 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Shape`、`Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 21/31 个代码块。

### Lines 108-109
````cpp
#endif
#elif defined(DECODE)
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `endif`, `elif`, `defined`, `DECODE` showing the main symbols being prepared or consumed here. It corresponds to block 22 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `endif`、`elif`、`defined`、`DECODE` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 22/31 个代码块。

### Lines 111-117
````cpp
#if PERSISTENT
#define NUM_SG _16
#define KV_TILE_SIZE _256
#else
#define NUM_SG _8
#define KV_TILE_SIZE _512
#endif
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `PERSISTENT`, `define`, `NUM_SG`, `_16` showing the main symbols being prepared or consumed here. It corresponds to block 23 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `PERSISTENT`、`define`、`NUM_SG`、`_16` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 23/31 个代码块。

### Lines 119-124
````cpp
#if HEAD_DIM == 16
  /* Tiny config for testing */
  using ShapeQK = Shape<_1, _16, _16>;       // (q,k,d)
  using ShapePV = Shape<_1, _16, _16>;       // (q,v,k)
  using ShapeOut = Shape<_1, _16>;           // (q,v)
  using SubgroupLayoutQK = Layout<Shape<_1, NUM_SG, _1>>;
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Shape`, `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 24 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Shape`、`Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 24/31 个代码块。

### Lines 126-130
````cpp
#elif HEAD_DIM == 64
    using ShapeQK = Shape<_1, KV_TILE_SIZE, _64>;
    using ShapePV = Shape<_1, _32, KV_TILE_SIZE>;
    using ShapeOut = Shape<_1, _64>;
    using SubgroupLayoutQK = Layout<Shape<_1, NUM_SG, _1>>;
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Shape`, `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 25 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Shape`、`Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 25/31 个代码块。

### Lines 132-136
````cpp
#elif HEAD_DIM == 96
    using ShapeQK = Shape<_1, KV_TILE_SIZE, _64>;
    using ShapePV = Shape<_1, _32, KV_TILE_SIZE>;
    using ShapeOut = Shape<_1, _96>;
    using SubgroupLayoutQK = Layout<Shape<_1, NUM_SG, _1>>;
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Shape`, `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 26 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Shape`、`Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 26/31 个代码块。

### Lines 138-142
````cpp
#elif HEAD_DIM == 128
    using ShapeQK = Shape<_1, KV_TILE_SIZE, _64>;
    using ShapePV = Shape<_1, _32, KV_TILE_SIZE>;
    using ShapeOut = Shape<_1, _128>;
    using SubgroupLayoutQK = Layout<Shape<_1, NUM_SG, _1>>;
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Shape`, `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 27 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Shape`、`Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 27/31 个代码块。

### Lines 144-152
````cpp
#elif HEAD_DIM == 192
    using ShapeQK = Shape<_1, KV_TILE_SIZE, _64>;
    using ShapePV = Shape<_1, _32, KV_TILE_SIZE>;
    using ShapeOut = Shape<_1, _192>;
    using SubgroupLayoutQK = Layout<Shape<_1, NUM_SG, _1>>;
#endif
#else
#error Either DECODE or PREFILL should be defined.
#endif
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `Shape`, `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 28 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `Shape`、`Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 28/31 个代码块。

### Lines 154-167
````cpp
#ifdef DECODE
  constexpr int PipelineStages = 1;
#else
  constexpr int PipelineStages = 2;
#endif
#ifdef IS_FLOAT_E5M2
  using ElementQ = cutlass::float_e5m2_t;
  using ElementK = cutlass::float_e5m2_t;
  using ElementV = cutlass::float_e5m2_t;
#elif defined(IS_FLOAT_E4M3)
  using ElementQ = cutlass::float_e4m3_t;
  using ElementK = cutlass::float_e4m3_t;
  using ElementV = cutlass::float_e4m3_t;
#else
````
**EN:** This block continues the file's flashattention / attention setup or compute path, with `ifdef`, `DECODE`, `constexpr`, `PipelineStages` showing the main symbols being prepared or consumed here. It corresponds to block 29 of 31 in the file order.
**CN:** 这一段继续推进本文件的FlashAttention / 注意力初始化或计算流程，其中 `ifdef`、`DECODE`、`constexpr`、`PipelineStages` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 29/31 个代码块。

### Lines 168-171
````cpp
  using ElementQ = bfloat16_t;
  using ElementK = bfloat16_t;
  using ElementV = bfloat16_t;
#endif
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `ElementQ`, `bfloat16_t`, `ElementK`, `ElementV` make the later flashattention / attention code easier to assemble and read. It corresponds to block 30 of 31 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `ElementQ`、`bfloat16_t`、`ElementK`、`ElementV` 这样的符号让后续FlashAttention / 注意力代码更容易组装和阅读。 它对应本文件顺序中的第 30/31 个代码块。

### Lines 173-179
````cpp
#if PERSISTENT
  return FMHAConfig<false, ShapeQK, ShapePV, ShapeOut, SubgroupLayoutQK, void, PipelineStages, /*persistent=*/true, ElementQ, ElementK, ElementV>::run(options);
#else
  return options.is_causal ? FMHAConfig<true, ShapeQK, ShapePV, ShapeOut, SubgroupLayoutQK, void, PipelineStages,  /*persistent=*/false, ElementQ, ElementK, ElementV>::run(options)
  : FMHAConfig<false, ShapeQK, ShapePV, ShapeOut, SubgroupLayoutQK, void, PipelineStages,  /*persistent=*/false, ElementQ, ElementK, ElementV>::run(options);
#endif
}
````
**EN:** This block finalizes a local computation or status path. The use of `Shape`, `Layout` helps conclude the current stage cleanly before the next block. It corresponds to block 31 of 31 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `Shape`、`Layout`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 31/31 个代码块。

## Key Concepts / 关键概念
- **EN:** Attention kernels coordinate Q/K/V shapes, scaling, and runner-side launch details.
  **CN:** 注意力内核需要协调 Q/K/V 形状、缩放因子以及 runner 侧启动细节。
- **EN:** The file mixes compile-time configuration with runtime problem sizes or arguments.
  **CN:** 该文件把编译期配置与运行期问题规模或参数结合起来。
- **EN:** Validation, benchmarking, or reporting code is interleaved with kernel setup.
  **CN:** 校验、基准测试或结果报告代码与内核配置过程交织在一起。

## Dependencies / 依赖关系
- **Direct dependencies / 直接依赖:** `xe_fmha_fwd_runner.hpp`
- **Runtime expectations / 运行时依赖:** CUTLASS/CUTE templates / CUTLASS/CUTE 模板
- **Build/context note / 构建说明:** The file relies on surrounding repository infrastructure, compile flags, and example utilities. / 该文件依赖仓库周边基础设施、编译选项以及示例辅助代码。
