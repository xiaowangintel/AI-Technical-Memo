# custom_mma.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/gemm/custom_mma.h`
**Purpose / 用途**: Compile-time adapter that replaces CUTLASS-selected MmaMultistage or MmaPipelined types with FMHA-specific custom implementations / 编译期适配层，将 CUTLASS 选出的 MmaMultistage 或 MmaPipelined 类型替换为 FMHA 专用实现
---
## Line-by-Line Analysis / 逐行分析
```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

#include "custom_mma_multistage.h"
#include "custom_mma_pipelined.h"
#include "cutlass/gemm/threadblock/mma_multistage.h"
#include "cutlass/gemm/threadblock/mma_pipelined.h"

template <typename Mma, int kMaxK>
struct MakeCustomMma;
```
**EN**: Lines 1-40. The header includes both custom implementations and the stock CUTLASS threadblock MMA types, then forward-declares MakeCustomMma. This establishes the pattern: callers provide an existing MMA type plus a K-limit, and template specialization picks the right replacement.
**CN**: 第1-40行：该头文件同时包含自定义实现和 CUTLASS 原生 threadblock MMA 类型，然后前置声明 MakeCustomMma。这表明了整体模式：调用方给出一个现成的 MMA 类型以及 K 上界，再由模板特化选择正确的替代实现。
```cpp

template <
    typename Shape,
    typename IteratorA,
    typename SmemIteratorA,
    cutlass::arch::CacheOperation::Kind CacheOpA,
    typename IteratorB,
    typename SmemIteratorB,
    cutlass::arch::CacheOperation::Kind CacheOpB,
    typename ElementC,
    typename LayoutC,
    typename Policy,
    int Stages,
    cutlass::gemm::SharedMemoryClearOption SharedMemoryClear,
    int kMaxK>
struct MakeCustomMma<
    cutlass::gemm::threadblock::MmaMultistage<
        Shape,
        IteratorA,
        SmemIteratorA,
        CacheOpA,
        IteratorB,
        SmemIteratorB,
        CacheOpB,
        ElementC,
        LayoutC,
        Policy,
        Stages,
        SharedMemoryClear>,
    kMaxK> {
  // Reduce the number of stages if we don't need that many
  static int constexpr kStages =
      kMaxK == cutlass::platform::numeric_limits<int>::max()
      ? Stages
      : cutlass::const_min(
            Stages,
            (kMaxK + int(Shape::kK) - 1) / int(Shape::kK));
  using Mma = cutlass::gemm::threadblock::CustomMmaMultistage<
      Shape,
      IteratorA,
      SmemIteratorA,
      CacheOpA,
      IteratorB,
      SmemIteratorB,
      CacheOpB,
      ElementC,
      LayoutC,
      Policy,
      kStages,
      SharedMemoryClear,
      kMaxK>;
};
```
**EN**: Lines 41-93. This partial specialization matches CUTLASS MmaMultistage. Its key logic is the compile-time kStages reduction: when kMaxK is bounded, it clamps the stage count to the minimum of the original stage setting and the number of K tiles actually needed. The output type is CustomMmaMultistage with the adjusted stage count and preserved iterators, cache operators, policy, and shared-memory clear mode.
**CN**: 第41-93行：这个偏特化匹配 CUTLASS 的 MmaMultistage。核心逻辑是编译期的 kStages 缩减：当 kMaxK 有上界时，它把 stage 数裁剪为“原始 stage 设置”和“实际需要的 K tile 数”中的较小值。输出类型是 CustomMmaMultistage，并保留了调整后的 stage 数以及原有迭代器、cache 操作、policy 和共享内存清零模式。
```cpp
template <
    typename Shape,
    typename IteratorA,
    typename SmemIteratorA,
    typename IteratorB,
    typename SmemIteratorB,
    typename ElementC,
    typename LayoutC,
    typename Policy,
    int kMaxK>
struct MakeCustomMma<
    cutlass::gemm::threadblock::MmaPipelined<
        Shape,
        IteratorA,
        SmemIteratorA,
        IteratorB,
        SmemIteratorB,
        ElementC,
        LayoutC,
        Policy>,
    kMaxK> {
  using Mma = cutlass::gemm::threadblock::CustomMmaPipelined<
      Shape,
      IteratorA,
      SmemIteratorA,
      IteratorB,
      SmemIteratorB,
      ElementC,
      LayoutC,
      Policy>;
};
```
**EN**: Lines 94-124. This specialization matches CUTLASS MmaPipelined and simply swaps in CustomMmaPipelined with the same template arguments. Unlike the multistage path, there is no stage-reduction logic because the pipelined implementation is fixed to a two-stage double buffer.
**CN**: 第94-124行：这个特化匹配 CUTLASS 的 MmaPipelined，并用相同模板参数直接替换为 CustomMmaPipelined。与 multistage 路径不同，这里没有 stage 缩减逻辑，因为 pipelined 实现固定为两级双缓冲。
---
## Key Concepts / 关键概念
- MakeCustomMma is a compatibility shim that lets higher-level code keep using CUTLASS-selected MMA types while redirecting execution to custom kernels. / MakeCustomMma 是一个兼容层，使上层代码仍可沿用 CUTLASS 选出的 MMA 类型，同时把执行重定向到自定义内核。
- The multistage specialization saves shared memory when the K dimension is statically bounded and does not require all original stages. / 当 K 维具有静态上界且不需要原始 stage 数时，multistage 特化会通过缩减 stages 来节省共享内存。
- The pipelined specialization is structurally simpler because its execution model is always the fixed two-stage pipeline. / pipelined 特化更简单，因为它的执行模型始终是固定的两级流水。
## Dependencies / 依赖项
- `custom_mma_multistage.h` — Defines the cp.async-backed multistage replacement used by FMHA / 定义 FMHA 使用的 cp.async 多级替代实现
- `custom_mma_pipelined.h` — Defines the classic two-stage pipelined replacement used by FMHA / 定义 FMHA 使用的经典两级流水替代实现
- `cutlass/gemm/threadblock/mma_multistage.h` — Identifies the stock CUTLASS multistage type to be pattern-matched / 指定需要模式匹配的 CUTLASS 原生 multistage 类型
- `cutlass/gemm/threadblock/mma_pipelined.h` — Identifies the stock CUTLASS pipelined type to be pattern-matched / 指定需要模式匹配的 CUTLASS 原生 pipelined 类型
