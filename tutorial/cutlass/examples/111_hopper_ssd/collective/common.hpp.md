# common.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/111_hopper_ssd/collective/common.hpp`  
**Purpose / 用途**: Shared SSD helper types, constants, and tensor utilities reused by the Hopper collective, kernel, and epilogue implementations. / 供 Hopper SSD collective、kernel 与 epilogue 复用的公共类型、常量和张量辅助工具。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-40 — Header prologue and namespace setup / 文件开头与命名空间设置
```cpp
/***************************************************************************************************
 * Copyright (c) 2025 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

#include "cutlass/kernel_hardware_info.h"
#include "cute/tensor.hpp"

namespace cutlass::ssd::collective {

using namespace cute;

```
**EN**: The file includes CuTe and CUTLASS building blocks needed by SSD collectives: GMMA traits, layout helpers, and debug utilities. It then opens `cutlass::ssd::collective`, signaling that these routines are shared implementation details for the real device kernel rather than standalone math kernels.
**CN**: 该文件引入了 SSD collective 所需的 CuTe/CUTLASS 基础组件：GMMA traits、布局辅助以及调试工具。随后进入 `cutlass::ssd::collective` 命名空间，表明这些函数是设备内核共享的底层实现细节，而不是独立的数学内核。

### Lines 41-68 — GMMA accumulation helpers / GMMA 累加控制辅助函数
```cpp
template<typename Atom, typename TA, typename TB, typename TC>
CUTE_DEVICE void gemm_reset_zero_acc(Atom& atom, TA const& tA, TB const& tB, TC&& tC) {
  constexpr int rA = decltype(rank(tA))::value;
  constexpr int rB = decltype(rank(tB))::value;
  constexpr int rC = decltype(rank(tC))::value;
  if constexpr (rA == 2 && rB == 2 && rC == 1) {
    CUTLASS_PRAGMA_UNROLL
    for (int k_block = 0; k_block < size<1>(tA); k_block++) {
      cute::gemm(atom, tA(_,k_block), tB(_,k_block), tC);
      atom.accumulate_ = GMMA::ScaleOut::One;
    }
  }
  else {
    static_assert(rA == 3 && rB == 3 && rC == 3);
    CUTLASS_PRAGMA_UNROLL
    for (int k_block = 0; k_block < size<2>(tA); k_block++) {
      cute::gemm(atom, tA(_,_,k_block), tB(_,_,k_block), tC);
      atom.accumulate_ = GMMA::ScaleOut::One;
    }
  }
}

template<typename Atom, typename TA, typename TB, typename TC>
CUTE_DEVICE void gemm_zero_acc(Atom& atom, TA const& tA, TB const& tB, TC&& tC) {
  atom.accumulate_ = GMMA::ScaleOut::Zero;
  gemm_reset_zero_acc(atom, tA, tB, tC);
}

```
**EN**: `gemm_reset_zero_acc()` iterates over K-blocks and invokes `cute::gemm()` while forcing later steps to accumulate with `ScaleOut::One`. `gemm_zero_acc()` is the entry helper that first sets `ScaleOut::Zero`, meaning the first GMMA call overwrites the accumulator and the rest accumulate into it.

This distinction matters in SSD kernels because many stages are multi-block reductions over `K`: the code must zero the accumulator exactly once, then preserve partial sums across subsequent GMMA instructions.
**CN**: `gemm_reset_zero_acc()` 会遍历 K-block，并调用 `cute::gemm()`；从第一次之后开始，把输出缩放模式切换为 `ScaleOut::One`，让后续步骤继续累加。`gemm_zero_acc()` 则是入口辅助函数：先把模式设为 `ScaleOut::Zero`，也就是第一次 GMMA 覆盖累加器，后面的 GMMA 都往里面加。

这在 SSD 内核里非常重要，因为很多阶段本质上都是跨多个 `K` 分块的归约：必须只在开始时清零一次累加器，之后保留已有部分和。

### Lines 69-97 — Converting MMA atoms to RS GMMA form / 将 MMA 原子转换为 RS 形式的 GMMA
```cpp
template<template<cute::GMMA::Major, cute::GMMA::Major, cute::GMMA::ScaleIn, cute::GMMA::ScaleIn> class Primitive, cute::GMMA::Major tA, cute::GMMA::Major tB, cute::GMMA::ScaleIn sA, cute::GMMA::ScaleIn sB>
inline auto __device__ constexpr convert_to_gmma_rs(cute::MMA_Atom<Primitive<tA, tB, sA, sB>> const& tiled_mma) {
    using Atom = cute::MMA_Atom<Primitive<tA, tB, sA, sB>>;
    using ElementA = typename Atom::ValTypeA;
    using ElementB = typename Atom::ValTypeB;
    using ElementC = typename Atom::ValTypeC;
    using Shape_MNK = typename Atom::Shape_MNK;
    using RS = decltype(cute::GMMA::rs_op_selector<ElementA, ElementB, ElementC, Shape_MNK, tA, tB, sA, sB>());
    return cute::MMA_Atom<RS>{};
}

template<template<cute::GMMA::ScaleIn, cute::GMMA::ScaleIn> class Primitive, cute::GMMA::ScaleIn sA, cute::GMMA::ScaleIn sB>
inline auto __device__ constexpr convert_to_gmma_rs(cute::MMA_Atom<Primitive<sA, sB>> const& tiled_mma) {
    using Atom = cute::MMA_Atom<Primitive<sA, sB>>;
    using ElementA = typename Atom::ValTypeA;
    using ElementB = typename Atom::ValTypeB;
    using ElementC = typename Atom::ValTypeC;
    using Shape_MNK = typename Atom::Shape_MNK;
    constexpr auto tA = cute::GMMA::Major::K;
    constexpr auto tB = cute::GMMA::Major::K;
    using RS = decltype(cute::GMMA::rs_op_selector<ElementA, ElementB, ElementC, Shape_MNK, tA, tB, sA, sB>());
    return cute::MMA_Atom<RS>{};
}

template<class Atom, class... Args>
CUTE_DEVICE auto constexpr convert_to_gmma_rs(cute::TiledMMA<Atom, Args...> const& tiled_mma) {
    return cute::TiledMMA<decltype(convert_to_gmma_rs(Atom{})), Args...>{};
}

```
**EN**: The overloaded `convert_to_gmma_rs()` helpers use `rs_op_selector()` to map a generic MMA atom or tiled MMA object to a row-stationary (RS) GMMA variant. In practice this lets the SSD kernel reuse the same high-level tensor code while selecting the Hopper instruction form that best matches the operand layout.
**CN**: 这一组重载的 `convert_to_gmma_rs()` 借助 `rs_op_selector()`，把通用的 MMA atom 或 tiled MMA 对象映射成 row-stationary（RS）形式的 GMMA。实际意义是：SSD 内核可以保持较高层次的张量代码不变，同时按操作数布局选择更合适的 Hopper 指令形态。

### Lines 98-109 — Layout reinterpretation helpers / 布局重解释辅助函数
```cpp
template<typename CLayout, typename AValueShape>
CUTE_DEVICE auto constexpr convert_c_layout_to_a_layout(CLayout const& c, AValueShape const& a) {
  return make_layout(
    make_shape(a, shape<1>(c), make_shape(shape<2>(c), size<0>(c) / size(a))),
    make_stride(stride<0>(c), stride<1>(c), make_stride(stride<2>(c), size<2>(a) * stride<0,2>(c))));
}

template<class Layout, class Stages = _1>
CUTE_DEVICE constexpr auto unstageSmemLayout(Layout const& layout, Stages stages = {}) {
    return composition(layout, make_tuple(_, _, make_layout(stages)));
}

```
**EN**: `convert_c_layout_to_a_layout()` reinterprets an accumulator-style `C` layout as an operand-style `A` layout by reshaping and restriding the fragment. `unstageSmemLayout()` removes pipeline staging structure from a shared-memory layout, which is useful when later code wants the logical tile shape rather than the staged storage view.
**CN**: `convert_c_layout_to_a_layout()` 通过重塑形状和步长，把偏向累加器的 `C` 布局重解释成偏向操作数的 `A` 布局。`unstageSmemLayout()` 则去掉共享内存布局中的流水 stage 结构，使后续代码拿到逻辑 tile 形状，而不是带 stage 维的存储视图。

### Lines 110-174 — Turning accumulator fragments into operand fragments / 把累加器片段转换为操作数片段
```cpp
template<class Element, class Accumulator, class OperandLayout_TV>
CUTE_DEVICE auto make_acc_into_op(Accumulator const& acc, OperandLayout_TV const& operand_layout_tv) {
  Tensor operand = make_fragment_like<Element>(convert_c_layout_to_a_layout(acc.layout(), shape<1>(operand_layout_tv)));
  Tensor operand_as_acc = make_tensor(operand.data(), acc.layout());

  cute::copy(acc, operand_as_acc);

  if constexpr (sizeof(Element) == 1) {
    
    // 00 11 22 33 00 11 22 33 acc layout
    // 00 00 11 11 22 22 33 33 operand layout
    // BB AA AA BB AA BB BB AA conflict-free exchange pattern
    //                         16-bit exchange; so process two at a time potentially
    int tid = threadIdx.x % 4;
    auto values_u32 = recast<uint32_t>(operand);

    CUTE_UNROLL
    for (int n = 0; n < size<1>(values_u32); n++) {
      CUTE_UNROLL
      for (int k = 0; k < size<2>(values_u32); k++) {
        CUTE_UNROLL
        for (int ii = 0; ii < 8; ii += 4) {

          uint32_t values_tmp_0 = values_u32(ii / 2 + 0, n, k);
          uint32_t values_tmp_1 = values_u32(ii / 2 + 1, n, k);

          // step A:
          // t 1 v 0 -> t 0 v 1
          // t 2 v 0 -> t 1 v 0
          // t 0 v 1 -> t 2 v 0
          // t 3 v 1 -> t 3 v 1

          int v_to_send = tid == 1 || tid == 2 ? 0 : 1;
          int v_to_recv = v_to_send;
          int t_to_recv_from = (0x3021 >> (tid * 4)) & 0xF;

          uint32_t values_tmp_a = v_to_send == 0 ? values_tmp_0 : values_tmp_1;

          values_tmp_a = __shfl_sync(0xFFFFFFFF, values_tmp_a, t_to_recv_from, 4);

          // step B:
          // t 0 v 0 -> t 0 v 0
          // t 3 v 0 -> t 1 v 1
          // t 1 v 1 -> t 2 v 1
          // t 2 v 1 -> t 3 v 0

          v_to_send = 1 - v_to_send;
          v_to_recv = 1 - v_to_recv;
          t_to_recv_from = (0x2130 >> (tid * 4)) & 0xF;

          uint32_t values_tmp_b = v_to_send == 0 ? values_tmp_0 : values_tmp_1;

          values_tmp_b = __shfl_sync(0xFFFFFFFF, values_tmp_b, t_to_recv_from, 4);

          values_u32(ii / 2 + 0, n, k) = __byte_perm(values_tmp_a, values_tmp_b, v_to_send == 0 ? 0x1054 : 0x5410);
          values_u32(ii / 2 + 1, n, k) = __byte_perm(values_tmp_a, values_tmp_b, v_to_send == 0 ? 0x3276 : 0x7632);
        }
      }
    }
  }

  return operand;
}

}  // namespace cutlass::fmha::collective
```
**EN**: `make_acc_into_op()` first allocates an operand fragment with the right logical layout, copies the accumulator values into it, and then applies a special path for 1-byte elements. The byte-sized path uses warp shuffles and `__byte_perm()` to rearrange packed lanes from GMMA accumulator order into the operand order expected by later tensor operations, while avoiding bank conflicts and wasted exchanges.

This is a hardware-oriented bridge rather than new SSD math: after a matrix multiply produces state or projection fragments, the kernel often needs to feed those results into a later stage as if they were ordinary operands.

The closing comment says `cutlass::fmha::collective`, but the actual namespace opened at the top is `cutlass::ssd::collective`; the mismatch is only in the comment.
**CN**: `make_acc_into_op()` 先创建一个逻辑布局正确的操作数片段，把累加器的值拷贝进去；如果元素是 1 字节类型，还会走一个特殊路径。这个字节级路径利用 warp shuffle 和 `__byte_perm()`，把 GMMA 累加器中的打包 lane 顺序改排成后续张量操作期望的操作数顺序，同时尽量避免 bank conflict 和无效交换。

这一步更像是面向硬件的数据桥接，而不是新的 SSD 数学：矩阵乘法得到状态或投影片段后，内核常常需要把结果再次当作“普通输入操作数”送入下一阶段。

文件末尾的注释写成了 `cutlass::fmha::collective`，但顶部真正打开的命名空间是 `cutlass::ssd::collective`；不一致只体现在注释中。

---

## Key Concepts / 关键概念

- GMMA zero-versus-accumulate control / GMMA 的清零与累加控制
- Row-stationary GMMA selection / Row-stationary GMMA 变体选择
- Layout reinterpretation between accumulator and operand views / 在累加器视图与操作数视图之间重解释布局
- Warp-level byte shuffle for packed fragments / 面向打包片段的 warp 级字节重排

## Dependencies / 依赖项

- `cute/tensor.hpp` — core tensor and layout abstractions / 核心张量与布局抽象
- `cute/atom/mma_traits_sm100.hpp` — GMMA trait machinery and instruction metadata / GMMA trait 机制与指令元数据
- `cute/util/debug.hpp` — debug printing utilities for CuTe objects / CuTe 对象的调试输出工具
