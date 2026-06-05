# reference_ssd.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/111_hopper_ssd/reference/reference_ssd.hpp`  
**Purpose / 用途**: Reference SSD implementation that reconstructs the chunked semiseparable algorithm on the host for correctness checking. / SSD 的参考实现：在主机端重建分块半可分算法，用于正确性校验。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-40 — Header, CuTe include, and phase macro / 文件头、CuTe 头文件与阶段宏
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

#include "cute/tensor.hpp"

// training or inference phase (not used yet)
// PHASE 0 : training
// PHASE 1 : inference
#define PHASE 0

```
**EN**: The file is intentionally lightweight: it only needs CuTe tensor support plus a `PHASE` macro placeholder. The comments mention training versus inference, but the current reference path does not branch on that macro yet.
**CN**: 这个文件有意保持轻量：只需要 CuTe 张量支持以及一个 `PHASE` 占位宏。注释提到了训练与推理阶段，但当前参考路径实际上还没有根据这个宏分支。

### Lines 41-70 — Scalar reference GEMM helper / 标量版参考 GEMM 辅助函数
```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////

template<
  bool transA,
  bool transB,
  class Element,
  class TensorA,
  class TensorB,
  class TensorC
>
void mma(
  TensorA tA,
  TensorB tB,
  TensorC tC) {
  using namespace cute;

  int M = transA ? int(shape<1>(tA)) : int(shape<0>(tA));
  int N = transB ? int(shape<1>(tB)) : int(shape<0>(tB));
  int K = transA ? int(shape<0>(tA)) : int(shape<1>(tA));
  for (int mi = 0; mi < M; ++mi) {
    for (int ni = 0; ni < N; ++ni) {
      for (int ki = 0; ki < K; ++ki) {
        float a = static_cast<float>(Element(transA ? tA(ki, mi) : tA(mi, ki)));
        float b = static_cast<float>(Element(transB ? tB(ki, ni) : tB(ni, ki)));
        tC(mi, ni) += a * b;
      }
    }
  }

}
```
**EN**: `mma()` is a tiny float-accumulating matrix multiply with optional transposition on each input. It is slow but transparent, which is exactly what a reference SSD implementation needs: each of the later “BMM” stages can be expressed with the same primitive and inspected directly.
**CN**: `mma()` 是一个使用 `float` 累加、支持两个输入独立转置的小型矩阵乘法。它虽然慢，但非常直观；这正是参考 SSD 实现所需要的，因为后面多个 “BMM” 阶段都可以统一表示成这个原语并直接检查。

### Lines 71-116 — Segment-sum matrix for intra-chunk recurrence / 用于 chunk 内递推的 segsum 矩阵
```cpp

template<
  class Element,
  class Tensor
>
auto segsum(Tensor tensor) {
  using namespace cute;
  auto C = shape<0>(tensor);
  auto L = shape<1>(tensor);
  auto cum_sum = make_tensor<float>(make_shape(C,L));
  // cum_sum
  for (int ci = 0; ci < C; ++ci) {
    for (int li = 0; li < L; ++li) {
      if (li == 0) {
        cum_sum(ci, li) = tensor(ci, li);
      }
      else {
        cum_sum(ci, li) = cum_sum(ci, li - 1) + tensor(ci, li);
      }
    }
  }
  auto seg_sum_out = make_tensor<float>(make_shape(C, L, L));
  // seg_sum
  // [      1,   0, 0]
  // [    e^y,   1, 0]
  // [e^(y+z), e^z, 1]
  CUTLASS_PRAGMA_UNROLL
  for (int ci = 0; ci < C; ++ci) {
    for (int i = 0; i < L; ++i) {
      for (int j = 0; j < L; ++j) {
        if (j < i) {
          float tmp = static_cast<float>(cum_sum(ci, i)) - static_cast<float>(cum_sum(ci, j));
          seg_sum_out(ci, i, j) = expf(tmp);
        } 
        else if (j == i) {
          seg_sum_out(ci, i, j) = 1.f;
        }
        else {
          seg_sum_out(ci, i, j) = 0.f;
        }
      }
    }
  }

  return seg_sum_out;
}
```
**EN**: `segsum()` first computes a prefix sum over each chunk row and then materializes a causal `L × L` matrix whose lower-triangular entries are exponentials of prefix differences. Concretely, for `j < i`, it emits `exp(prefix[i] - prefix[j])`; on the diagonal it emits `1`, and for `j > i` it emits `0`.

This is the semiseparable structure in explicit form: every earlier position `j` contributes to later position `i` through an exponential decay determined by the interval sum of `DeltaA`.
**CN**: `segsum()` 先对每个 chunk 行做前缀和，再显式构造一个满足因果性的 `L × L` 矩阵，其中下三角元素是前缀差的指数。具体来说，当 `j < i` 时输出 `exp(prefix[i] - prefix[j])`；对角线上输出 `1`；当 `j > i` 时输出 `0`。

这就是半可分结构的显式写法：较早位置 `j` 对较晚位置 `i` 的影响，会通过由 `DeltaA` 区间和决定的指数衰减来传递。

### Lines 117-156 — Cumsum-derived factors for inter-chunk recurrence / 为 chunk 间递推准备的 cumsum 因子
```cpp

template<
  class Element,
  class Tensor
>
auto cumsum(
  Tensor tensor) {
  using namespace cute;
  auto C = shape<0>(tensor);
  auto L = shape<1>(tensor);
  auto cum_sum              = make_tensor<float>(make_shape(C,L));
  auto cum_sum_out          = make_tensor<Element>(make_shape(C,L));
  auto cum_sum_exp_out      = make_tensor<float>(make_shape(C, L));
  auto cum_sum_exp_out_last = make_tensor<float>(make_shape(C, L));
  auto last_column          = make_tensor<float>(make_shape(C));
  // [x, x+y, x+y+z, ..]
  CUTLASS_PRAGMA_UNROLL
  for (int ci = 0; ci < C; ++ci) {
    for (int li = 0; li < L; ++li) {
      if (li == 0) {
        cum_sum(ci, li) = tensor(ci, li);
      }
      else {
        cum_sum(ci, li) = cum_sum(ci, li - 1) + tensor(ci, li);
      }
      // cum_sum_out(ci, li) = static_cast<Element>(cum_sum(ci, li));
    }
  }
  CUTLASS_PRAGMA_UNROLL
  for (int ci = 0; ci < C; ++ci) {
    last_column(ci) = static_cast<float>(cum_sum(ci, L-1));
    CUTLASS_PRAGMA_UNROLL
    for (int li = 0; li < L; ++li) {
      cum_sum_exp_out_last(ci, li) = expf(static_cast<float>(last_column(ci) - cum_sum(ci, li)));
      cum_sum_exp_out(ci, li) = expf(static_cast<float>(cum_sum(ci, li)));
    }
  }

  return make_tuple(cum_sum_exp_out_last, last_column, cum_sum_exp_out);
}
```
**EN**: `cumsum()` computes three related quantities from the same prefix sums: `exp(last - prefix[l])`, the final prefix value `last`, and `exp(prefix[l])`. Together they let the reference code express chunk-to-chunk state passing and within-chunk output rescaling without repeatedly multiplying long chains of exponentials.
**CN**: `cumsum()` 从同一组前缀和中生成三个相关量：`exp(last - prefix[l])`、最终前缀值 `last`，以及 `exp(prefix[l])`。它们组合起来，使参考代码能够表达 chunk 到 chunk 的状态传递，以及 chunk 内输出的重缩放，而不必反复显式相乘很长的指数链。

### Lines 157-190 — Reference kernel signature and tensor conventions / 参考内核签名与张量约定
```cpp

template<
  bool  HAS_D,
  bool  D_HAS_HDIM,
  bool  HAS_Z,
  class TensorY,
  class TensorF,
  class TensorX,
  class TensorDelta,
  class TensorDeltaA,
  class TensorB,
  class TensorC,
  class TensorD,
  class TensorZ,
  class Params
>
void ssd_reference_impl(
    TensorY mY, TensorF mF,
    TensorX mX, TensorDelta mDelta, TensorDeltaA mDeltaA,
    TensorB mB, TensorC mC, TensorD mD, TensorZ mZ,
    Params params) {

  using namespace cute;
  using Element = typename Params::Element;
  using ElementAcc = typename Params::ElementAcc;

  // x       [b, eh, d, c, l]
  // delta   [b, eh, c, l]
  // delta_A [b, eh, c, l]
  // B       [b,  g, n, c, l]
  // C       [b,  g, n, c, l]
  // y       [b, eh, d, c, l]
  // fstate  [b, eh, d, n]
  // d       [   eh, d]
```
**EN**: `ssd_reference_impl()` is parameterized by three feature flags: whether to add a direct term `D`, whether `D` is per hidden dimension or scalar per head, and whether to apply a gate `Z`. The tensor layout comments are important because they connect the code to the SSD decomposition: `c` indexes chunks, `l` indexes positions inside each chunk, `d` is hidden width, and `n` is state width.
**CN**: `ssd_reference_impl()` 由三个特性开关参数化：是否加入直通项 `D`、`D` 是按隐藏维变化还是每个 head 一个标量，以及是否应用门控 `Z`。这里的张量布局注释非常关键，因为它把代码和 SSD 分解一一对应起来：`c` 表示 chunk，`l` 表示 chunk 内位置，`d` 是隐藏维，`n` 是状态维。

### Lines 191-212 — Batch/head loop setup and IntraBMM1 / batch/head 循环设置与 IntraBMM1
```cpp
  auto [G, B, EH, C, L, D, N] = params.get_problem_shape();
  int group_ratio = EH / G;
  for (int b = 0; b < B; ++b) {
    for (int eh = 0; eh < EH; ++eh) {
      int g = eh / group_ratio;
      auto tY      = mY(b,eh,_,_,_);
      auto tF      = mF(b,eh,_,_);
      auto tX      = mX(b,eh,_,_,_);
      auto tDelta  = mDelta(b,eh,_,_);
      auto tDeltaA = mDeltaA(b,eh,_,_);
      auto tB      = mB(b,g,_,_,_);
      auto tC      = mC(b,g,_,_,_);
      auto tD      = mD(eh,_);
      auto tZ      = mZ(b,eh,_,_,_);
      // IntraBMM1 BxC, LxLxN, NT
      // B: [n, c, l]
      // C: [n, c, l]
      // O: [c, l, l]
      auto tIntraBMM1_out = make_tensor<float>(make_shape(C,L,L));
      for (int ci = 0; ci < C; ++ci) {
        mma<true,true,Element>(tC(_,ci,_), tB(_,ci,_), tIntraBMM1_out(ci,_,_)); 
      }
```
**EN**: The outer loops iterate over batch and expanded heads, then derive the group index `g`. `IntraBMM1` computes `C^T B` for each chunk, producing an `L × L` interaction matrix per chunk row.

This matrix is the chunk-local analogue of attention weights, except it is part of the SSD semiseparable factorization rather than a softmax attention kernel.
**CN**: 最外层循环遍历 batch 和扩展后的 heads，然后据此推导 group 索引 `g`。`IntraBMM1` 对每个 chunk 计算 `C^T B`，从而为每个 chunk 行生成一个 `L × L` 的相互作用矩阵。

这个矩阵可以看作 chunk 内部的“局部交互权重”，但它属于 SSD 的半可分分解，而不是 softmax 注意力。

### Lines 213-233 — Applying segsum and forming IntraBMM2 / 应用 segsum 并形成 IntraBMM2
```cpp
      // Pre_IntraBMM2 DeltaA_IntraBMM2 x Delta x IntraBMM_out
      // DeltaA_xxx   : [c, l, l]
      // Delta        : [c, l, _]
      // IntraBMM1_out: [c, l, l]
      auto tDeltaA_IntraBMM2 = segsum<Element>(tDeltaA);
      auto tIntraBMM2_inp    = make_tensor<float>(make_shape(C, L, L));
      for (int ci = 0; ci < C; ++ci) {
        for (int i = 0; i < L; ++i) {
          for (int j = 0; j < L; ++j) {
            tIntraBMM2_inp(ci, i, j) = tDeltaA_IntraBMM2(ci, i, j) * tDelta(ci, j) * tIntraBMM1_out(ci, i, j);
          }
        }
      }
      // IntraBMM2 IntraBMM2_inp x X, LxDxL, TT
      // IntraBMM2_inp: [c, l, l]
      // X            : [d, c, l]
      // IntraBMM2_out: [c, l, d]
      auto tIntraBMM2_out = make_tensor<float>(make_shape(C,L,D));
      for (int ci = 0; ci < C; ++ci) {
        mma<false,false,Element>(tIntraBMM2_inp(ci,_,_), tX(_,ci,_), tIntraBMM2_out(ci,_,_)); 
      }
```
**EN**: `segsum(DeltaA)` supplies the lower-triangular decay matrix, and the code multiplies it elementwise with `Delta` and `IntraBMM1_out`. The resulting `IntraBMM2_inp` is then multiplied with `X` to produce `IntraBMM2_out`, i.e. the purely chunk-local contribution to the final output.
**CN**: `segsum(DeltaA)` 提供了下三角衰减矩阵，代码再把它与 `Delta` 和 `IntraBMM1_out` 做逐元素相乘。得到的 `IntraBMM2_inp` 再与 `X` 相乘，形成 `IntraBMM2_out`，也就是最终输出中的纯 chunk 内贡献部分。

### Lines 234-254 — Preparing and computing InterBMM1 / 准备并计算 InterBMM1
```cpp
      // Pre_InterBMM1 DeltaA_InterBMM1 x Delta x B
      // DeltaA_xxx   : [c, l]
      // Delta        : [c, l]
      // IntraBMM1_out: [c, n, l]
      auto [tDeltaA_InterBMM1, tLast, tCumsum_exp] = cumsum<Element>(tDeltaA); 
      auto tInterBMM1_inp    = make_tensor<float>(make_shape(C, N, L));
      for (int ci = 0; ci < C; ++ci) {
        for (int i = 0; i < N; ++i) {
          for (int j = 0; j < L; ++j) {
            tInterBMM1_inp(ci, i, j) = tDeltaA_InterBMM1(ci, j) * tDelta(ci, j) * tB(i, ci, j);
          }
        }
      }
      // InterBMM1 InterBMM1_inp x X, NxDxL, swapAB, TT
      // InterBMM1_inp: [c, n, l]
      // X            : [d, c, l]
      // InterBMM1_out: [c, n, d]
      auto tInterBMM1_out = make_tensor<float>(make_shape(C,N,D));
      for (int ci = 0; ci < C; ++ci) {
        mma<false,false,Element>(tInterBMM1_inp(ci,_,_), tX(_,ci,_), tInterBMM1_out(ci,_,_));
      }
```
**EN**: The call to `cumsum(DeltaA)` yields three factors, and the first one—`exp(last - prefix)`—is used to scale `Delta * B` before multiplying by `X`. The resulting `InterBMM1_out[c, n, d]` is a per-chunk state contribution living in the latent state dimension `n` rather than the output dimension `l`.
**CN**: 这里调用 `cumsum(DeltaA)` 得到三个因子，其中第一个 `exp(last - prefix)` 会用来缩放 `Delta * B`，再与 `X` 相乘。结果 `InterBMM1_out[c, n, d]` 表示每个 chunk 的状态贡献，它位于潜在状态维 `n` 上，而不是最终输出位置维 `l` 上。

### Lines 255-279 — Chunk-to-chunk state recurrence and InterBMM2 / chunk 间状态递推与 InterBMM2
```cpp
      // Initialize state
      // PreInterBMM2 
      // InterBMM1_out: [c, n, d]
      // Last         : [c] 
      auto tInterBMM2_inp      = make_tensor<float>(make_shape(C, N, D));
      for (int ci = 0; ci < C; ++ci) {
        for (int ni = 0; ni < N; ++ ni){
          for (int di = 0; di < D; ++di) {
            if (ci == 0) {
              tInterBMM2_inp(ci, ni, di) = 0;
            }
            else {
              tInterBMM2_inp(ci, ni, di) = tInterBMM1_out(ci - 1, ni, di) + expf(tLast(ci - 1)) * tInterBMM2_inp(ci - 1, ni, di);
            }
          }
        }
      }
      // InterBMM2 InterBMM2_inp x C, LxDxN, NT
      // C            : [n, c, l]
      // InterBMM2_inp: [c, n, d]
      // InterBMM2_out: [c, l, d]
      auto tInterBMM2_out = make_tensor<float>(make_shape(C,L,D));
      for (int ci = 0; ci < C; ++ci) {
        mma<true,true,Element>(tC(_,ci,_), tInterBMM2_inp(ci,_,_), tInterBMM2_out(ci,_,_));
      }
```
**EN**: `tInterBMM2_inp` is built with an explicit recurrence over chunks: the previous chunk’s freshly computed state contribution is added to the old carried state after multiplying by `exp(last)`. This is the chunk-level state passing step of SSD/Mamba2.

`InterBMM2` then projects that carried state back through `C`, yielding the inter-chunk contribution in output space. In other words, the code separates “update the latent state” from “read the latent state out”.
**CN**: `tInterBMM2_inp` 通过显式的跨 chunk 递推来构造：上一 chunk 新计算出的状态贡献，会与旧的携带状态（先乘上 `exp(last)`）相加。这正是 SSD/Mamba2 中 chunk 级状态传递的核心步骤。

随后的 `InterBMM2` 再用 `C` 把携带状态投影回输出空间，得到 chunk 间的输出贡献。也就是说，代码把“更新潜在状态”和“读取潜在状态”两个动作明确分开了。

### Lines 280-315 — Final output synthesis and final state writeback / 最终输出合成与最终状态回写
```cpp
      // Epilogue Cumsum_exp x InterBMM2_out + IntraBMM2_out
      // InterBMM2_out: [c, l, d]
      // IntraBMM2_out: [c, l, d]
      // Cumsum_exp   : [c, l]
      for (int ci = 0; ci < C; ++ci) {
        for (int li = 0; li < L; ++li) {
          for (int di = 0; di < D; ++di) {
            float y = tInterBMM2_out(ci, li, di) * tCumsum_exp(ci, li) + tIntraBMM2_out(ci, li, di);
            float scale;
            if constexpr (D_HAS_HDIM) {
              scale = static_cast<float>(tD(di));
            }
            else {
              scale = static_cast<float>(tD(_0{}));
            }
            if constexpr (HAS_D) {
              y = y + static_cast<float>(tX(di, ci, li)) * scale;
            }
            else {
              y = y;
            }
            if constexpr (HAS_Z) {
              float z = static_cast<float>(tZ(di, ci, li));
              // y = y * z * (1 / (1 + exp(-z)));
              y = y * z * (1 / (1 + exp(-z)));
            }
            tY(di, ci, li) = static_cast<typename Params::Element>(y);
          }
        }
      }
      // Epilogue Fstate(last C)
      for (int ni = 0; ni < N; ++ ni){
        for (int di = 0; di < D; ++di) {
          tF(di, ni) = static_cast<typename Params::Element>(tInterBMM1_out(C - 1, ni, di) + expf(tLast(C - 1)) * tInterBMM2_inp(C - 1, ni, di));
        }
      }
```
**EN**: For each `(c, l, d)`, the code adds the chunk-local contribution `IntraBMM2_out` to the inter-chunk contribution `InterBMM2_out * exp(prefix)`. It then optionally adds the direct term `x * D` and optionally multiplies by `z * sigmoid(z)`, which is a SiLU-style gate.

The final-state tensor `F` stores the state that would be passed to the next chunk or next decode step. This is why SSD can be used in streaming or recurrent settings: the expensive history is summarized into a compact state vector.
**CN**: 对每个 `(c, l, d)`，代码都会把 chunk 内贡献 `IntraBMM2_out` 与 chunk 间贡献 `InterBMM2_out * exp(prefix)` 相加。随后，它还会按需加入直通项 `x * D`，并按需乘上 `z * sigmoid(z)`，也就是一种 SiLU 风格的门控。

最终状态张量 `F` 保存的是可传给下一个 chunk 或下一次解码步骤的状态。这正是 SSD 能用于流式或递归场景的原因：昂贵的历史被压缩成了一个紧凑的状态向量。

### Lines 316-345 — Wrapper entry point and file end / 包装入口与文件结尾
```cpp
    }
  }
}

/////////////////////////////////////////////////////////////////////////////////////////////////

template<
  bool  HAS_D,
  bool  D_HAS_HDIM,
  bool  HAS_Z,
  class TensorY,
  class TensorF,
  class TensorX,
  class TensorDelta,
  class TensorDeltaA,
  class TensorB,
  class TensorC,
  class TensorD,
  class TensorZ,
  class Params
>
void ssd_reference(
    TensorY mY, TensorF mF,
    TensorX mX, TensorDelta mDelta, TensorDeltaA mDeltaA,
    TensorB mB, TensorC mC, TensorD mD, TensorZ mZ,
    Params params) {
  ssd_reference_impl<HAS_D, D_HAS_HDIM, HAS_Z>(mY, mF, mX, mDelta, mDeltaA, mB, mC, mD, mZ, params);
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: `ssd_reference()` is just a thin wrapper over `ssd_reference_impl()`. The extra layer keeps the call site simple while leaving the feature-flag-rich implementation in one templated function.
**CN**: `ssd_reference()` 只是 `ssd_reference_impl()` 的一层很薄的包装。多这一层的目的是让调用点保持简洁，而把真正带有多个特性开关的实现集中放在一个模板函数中。

---

## Key Concepts / 关键概念

- Semiseparable lower-triangular decay structure / 半可分下三角衰减结构
- Chunk-local versus inter-chunk decomposition / chunk 内与 chunk 间分解
- Prefix-scan factors for state passing / 用于状态传递的前缀扫描因子
- Final latent state output for streaming inference / 面向流式推理的最终潜在状态输出
- Optional direct term `D` and gate `Z` / 可选的直通项 `D` 与门控 `Z`

## Dependencies / 依赖项

- `cute/tensor.hpp` — provides the tensor views and slicing used by the reference code / 为参考代码提供张量视图与切片能力
