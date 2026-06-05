# reference_ssd.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/112_blackwell_ssd/reference/reference_ssd.hpp`  
**Purpose / 用途**: Reference SSD implementation for Blackwell that reconstructs the chunked semiseparable algorithm and validates outputs. / Blackwell SSD 的参考实现：重建分块半可分算法并用于输出校验。

---

## Line-by-Line Analysis / 逐行分析

### 1. Lines 1-52 | Header setup and namespace opening | 头文件设置与命名空间开启

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

/////////////////////////////////////////////////////////////////////////////////////////////////

template<
  bool transA,
  bool transB,
  class Element,
  class TensorA,
  class TensorB,
  class TensorC
>
void mma(TensorA tA, TensorB tB, TensorC tC) {
  using namespace cute;
```

- **EN:** The file begins with standard boilerplate plus the includes needed for CuTe tensors, debug helpers, and CUTLASS host utilities. It then opens `cutlass::ssd::kernel`, the same namespace used by the example kernels.

- **CN:** 文件首先给出标准样板和 CuTe 张量、调试辅助以及 CUTLASS 主机工具所需的头文件，然后打开 `cutlass::ssd::kernel` 命名空间，与示例 kernel 保持一致。

### 2. Lines 53-68 | Naive matrix multiply helper | 朴素矩阵乘辅助函数

```cpp

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

- **EN:** `mma` is a simple triply nested loop that supports optional transposition of both operands. It is intentionally scalar and slow, but perfect for expressing the mathematical intent of the reference path.

- **CN:** `mma` 是一个三重循环的朴素矩阵乘辅助函数，并支持两个输入矩阵各自可选转置。它刻意保持标量且速度不快，但非常适合表达参考路径的数学含义。

### 3. Lines 69-113 | `segsum`: build the lower-triangular intra-chunk decay matrix | `segsum`：构造块内下三角衰减矩阵

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

- **EN:** This routine first computes chunk-local cumulative sums and then expands them into a lower-triangular matrix with entries `exp(cumsum[i] - cumsum[j])` for `j < i`, `1` on the diagonal, and `0` above it. That matrix is the diagonal block operator for chunked SSD.

- **CN:** 该例程先计算 chunk 内局部累计和，再把它展开为一个下三角矩阵：当 `j < i` 时条目为 `exp(cumsum[i] - cumsum[j])`，对角线为 `1`，上三角为 `0`。这个矩阵就是分块 SSD 的对角块算子。

### 4. Lines 114-152 | `cumsum`: boundary decays and prefix exponentials | `cumsum`：边界衰减与前缀指数

```cpp

template<
  class Element,
  class Tensor
>
auto cumsum(Tensor tensor) {
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

- **EN:** The second helper produces three objects from `DeltaA`: the decay from each position to the chunk end, the last cumulative value per chunk, and `exp(prefix)` for every position. These are exactly the ingredients needed for chunk-state updates and final output rescaling.

- **CN:** 第二个辅助函数从 `DeltaA` 中生成三个对象：每个位置到 chunk 末尾的衰减、每个 chunk 的最终累计值，以及每个位置的 `exp(prefix)`。这些正是做 chunk 状态更新和最终输出缩放所需的关键量。

### 5. Lines 153-180 | Reference kernel signature and tensor conventions | 参考 kernel 签名与张量约定

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
```

- **EN:** The main reference function is heavily templated so it can model optional `D`/`Z` behavior while staying generic over tensor engines/layouts. The comments establish the exact tensor conventions used everywhere else in the example.

- **CN:** 主参考函数使用了大量模板参数，因此既能表达可选的 `D`/`Z` 行为，也能对不同 tensor engine/layout 保持泛化。注释则明确给出了后续整个示例共享的张量约定。

### 6. Lines 181-229 | Per-head slicing plus intra-chunk diagonal work | 按头切片并计算块内对角部分

```cpp
  // delta_A [b, eh, c, l]
  // B       [b,  g, n, c, l]
  // C       [b,  g, n, c, l]
  // y       [b, eh, d, c, l]
  // fstate  [b, eh, d, n]
  // d       [   eh, d]
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

- **EN:** For each batch/head pair, the code derives the owning group, slices the tensors, computes `C^T B`-style intra-chunk interactions, multiplies by the segment-sum decay and `Delta`, and finally projects that chunk-local result through `X`. This is the diagonal part of the semiseparable decomposition.

- **CN:** 对于每个 batch/head 对，代码先推导所属 group，再切出对应张量，计算类似 `C^T B` 的块内交互，然后乘上 segment-sum 衰减和 `Delta`，最后再通过 `X` 完成投影。这一部分对应半可分分解中的对角块。

### 7. Lines 230-275 | Inter-chunk state preparation and recurrence | 块间状态准备与递推

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

- **EN:** The code next forms the right-factor state contribution with `B`, `Delta`, and the chunk-end decay from `cumsum`, then propagates those states chunk by chunk using `exp(last)` as the recurrence multiplier. This is the off-diagonal middle term that carries information across chunk boundaries.

- **CN:** 随后，代码利用 `B`、`Delta` 和 `cumsum` 给出的 chunk 末端衰减构造右因子状态贡献，再用 `exp(last)` 作为递推乘子逐 chunk 传播状态。这正是跨 chunk 携带信息的非对角中间项。

### 8. Lines 276-314 | Output epilogue and final state emission | 输出后处理与最终状态写出

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
    }
  }
}
```

- **EN:** Here the left factor `C` maps propagated states back to outputs, `exp(prefix)` restores the correct position-wise decay, and the diagonal/off-diagonal pieces are added together. The epilogue then optionally adds the residual `D` term, optionally gates with `Z`, and writes both `Y` and the final recurrent state `F`.

- **CN:** 这里左因子 `C` 把传播后的状态重新映射回输出，`exp(prefix)` 恢复每个位置应有的衰减，然后把对角项与非对角项相加。接着后处理可选地加入残差 `D` 项、可选地施加 `Z` 门控，并最终写出 `Y` 与最终递归状态 `F`。

### 9. Lines 315-341 | Thin wrapper and namespace close | 薄封装与命名空间结束

```cpp

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

- **EN:** The public `ssd_reference` wrapper simply forwards to `ssd_reference_impl`. Keeping the wrapper separate makes it easy to preserve a clean call site while reserving space for future specializations.

- **CN:** 公开的 `ssd_reference` 包装函数只是直接转发到 `ssd_reference_impl`。把包装层单独保留出来，有助于保持调用点简洁，也为未来特化实现预留了空间。

## Takeaways | 总结

- **EN:** This file is the clearest mathematical description of the example: it shows how chunked scan, state recurrence, and semiseparable factorization combine to realize SSD/Mamba2.

- **CN:** 这个文件是整个示例中最清晰的数学说明：它展示了分块扫描、状态递推和半可分分解如何组合起来实现 SSD/Mamba2。

---

## Key Concepts / 关键概念

- **Chunked semiseparable decomposition / 分块半可分分解**

  - **EN:** The sequence is partitioned into `C` chunks of length `L`, letting the algorithm treat diagonal intra-chunk blocks separately from off-diagonal chunk-to-chunk interactions.

  - **CN:** 序列被划分为 `C` 个长度为 `L` 的 chunk，从而把块内对角部分与块间的非对角交互分开处理。

- **Segment-sum decay matrix / 分段求和衰减矩阵**

  - **EN:** `segsum` builds a lower-triangular matrix of exponentiated cumulative `A` differences. That matrix is the diagonal-block decay operator used inside each chunk.

  - **CN:** `segsum` 构造由累计 `A` 差值指数化后形成的下三角矩阵，它就是每个 chunk 内部使用的对角块衰减算子。

- **State updates across chunks / 跨 chunk 的状态更新**

  - **EN:** `cumsum` returns both `exp(prefix)` and `exp(last-prefix)` forms so the code can propagate recurrent state from one chunk boundary to the next.

  - **CN:** `cumsum` 同时返回 `exp(prefix)` 与 `exp(last-prefix)` 两种形式，使代码能够把递归状态从一个 chunk 边界传播到下一个边界。

- **Low-rank off-diagonal factorization / 低秩非对角分解**

  - **EN:** The reference organizes off-diagonal work into a right factor (`B`), middle recurrence (`A` decay), and left factor (`C`), matching the semiseparable viewpoint used by SSD/Mamba2.

  - **CN:** 参考实现把非对角部分组织成右因子（`B`）、中间递推（`A` 衰减）和左因子（`C`），对应 SSD/Mamba2 的半可分视角。

- **Optional residual/gating epilogue / 可选残差与门控后处理**

  - **EN:** After combining intra/inter-chunk contributions, the code can add the `D` skip term and optionally apply the `Z` gate, though this Blackwell example keeps `Z` disabled.

  - **CN:** 在合并块内/块间贡献后，代码可以继续加上 `D` 跳连项，并可选施加 `Z` 门控，不过这个 Blackwell 示例默认关闭了 `Z`。

## Dependencies / 依赖项

- **`cute/tensor.hpp`**

  - **EN:** All temporary tensors, slicing operations, and shape utilities come from CuTe.

  - **CN:** 所有临时张量、切片操作和形状工具都来自 CuTe。

- **The `Params` interface from `Options`**

  - **EN:** The reference expects `params.get_problem_shape()` and type aliases like `Element` / `ElementAcc`, so it is tightly coupled to the example’s option object.

  - **CN:** 参考实现依赖 `params.get_problem_shape()` 以及 `Element` / `ElementAcc` 之类的类型别名，因此与示例中的选项对象紧密耦合。

- **Input/output tensor layout contract**

  - **EN:** The code assumes the logical layouts documented in comments: `X[b,eh,d,c,l]`, `B[b,g,n,c,l]`, `F[b,eh,d,n]`, and so on.

  - **CN:** 代码假定注释中说明的逻辑布局成立，例如 `X[b,eh,d,c,l]`、`B[b,g,n,c,l]`、`F[b,eh,d,n]` 等。
