# moe_fp8.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/moe_fp8.cpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Runtime integration and dispatch
```cpp
#include "common.h"
#include "gemm.h"
#include "moe.h"

template <typename scalar_t>
void fused_experts_fp8_kernel_impl(
    scalar_t* __restrict__ output,
    scalar_t* __restrict__ ic0,
    scalar_t* __restrict__ ic1,
    scalar_t* __restrict__ ic2,
    scalar_t* __restrict__ A_tmp,
    scalar_t* __restrict__ B_tmp,
    float* __restrict__ C_tmp,
    const scalar_t* __restrict__ input,
    const at::Float8_e4m3fn* __restrict__ packed_w1,
    const at::Float8_e4m3fn* __restrict__ packed_w2,
    const float* __restrict__ w1s,
    const float* __restrict__ w2s,
    int64_t block_size_N,
    int64_t block_size_K,
    const float* __restrict__ topk_weights,
    const int32_t* __restrict__ sorted_ids,
    const int32_t* __restrict__ expert_ids,
    const int32_t* __restrict__ offsets,
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 25-48: Runtime integration and dispatch
```cpp
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t E,
    int64_t topk,
    int64_t num_tokens_post_pad) {
  constexpr int64_t BLOCK_M = block_size_m();
  constexpr int64_t BLOCK_N = block_size_n();

  // stage 1: intermediate_cache0 = hidden_states @ w1
  const int64_t MB = div_up(num_tokens_post_pad, BLOCK_M);
  const int64_t NB = div_up(2 * N, BLOCK_N);
  int64_t scale_size_N = div_up(2 * N, block_size_N);
  int64_t scale_size_K = div_up(K, block_size_K);
  int64_t blocks_n_per_group = block_size_N / BLOCK_N;

  const int64_t stride_e = 2 * N * K;
  const int64_t stride_n = K;

  int64_t avg_M = std::max(int64_t(1), M * topk / E);
  const bool use_brgemm = can_use_brgemm<at::Float8_e4m3fn>(avg_M);

  int64_t B_tmp_size_per_thread = MAX_CACHE_BLOCK_SIZE * BLOCK_N * std::max(K, N);
```
**EN:** This section uses `block_size_m`, `block_size_n`, `div_up` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`block_size_m`、`block_size_n`、`div_up`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 49-69: Runtime integration and dispatch
```cpp
  // here we only parallel on half of 2N to fuse silu_and_mul with gemm
  parallel_2d(MB, NB, [&](int64_t mb0, int64_t mb1, int64_t nb0, int64_t nb1) {
    // get local pointers
    int tid = get_thread_num();
    scalar_t* __restrict__ A = A_tmp + tid * BLOCK_M * K;

    loop_2d<at::Float8_e4m3fn>(mb0, mb1, nb0, nb1, BLOCK_N * K, [&](int64_t mb, int64_t nb, int64_t nb_offset) {
      int64_t n_size = std::min(2 * N - nb * BLOCK_N, BLOCK_N);

      // B shape [K, n_size] in vnni format
      int32_t expert_id = expert_ids[mb];
      const at::Float8_e4m3fn* __restrict__ B = packed_w1 + expert_id * stride_e + nb * BLOCK_N * stride_n;
      const float* __restrict__ Bs =
          w1s + expert_id * scale_size_N * scale_size_K + (nb / blocks_n_per_group) * scale_size_K;

      // do unpacking for the first row or a new expert
      int32_t pre_expert_id = mb == 0 ? -1 : expert_ids[mb - 1];
      bool do_unpack = (mb == mb0) || (expert_id != pre_expert_id);

      int64_t m_size = offsets[mb + 1] - offsets[mb];
```
**EN:** This section uses `parallel_2d`, `get_thread_num`, `min` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`parallel_2d`、`get_thread_num`、`min`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 70-97: Control flow and branching
```cpp
      if (nb_offset == 0) {
        // 1.a load A
        const int32_t* A_ids = sorted_ids + mb * BLOCK_M;
        for (int64_t m = 0; m < m_size; ++m) {
          int32_t index = A_ids[m] / topk;
          copy_stub(A + m * K, input + index * K, K);
        }
      }

      const int64_t offset = offsets[mb];
      tinygemm_kernel<scalar_t>(
          /*   A            */ A,
          /*   B            */ B,
          /*   C            */ ic0 + offset * 2 * N + nb * BLOCK_N,
          /*   Btmp         */ B_tmp + tid * B_tmp_size_per_thread + nb_offset * BLOCK_N * K,
          /*   Ctmp         */ C_tmp + tid * 2 * BLOCK_M * BLOCK_N,
          /*   scale        */ Bs,
          /*   M            */ m_size,
          /*   N            */ n_size,
          /*   K            */ K,
          /*   lda          */ K,
          /*   ldb          */ n_size,
          /*   ldc          */ 2 * N,
          /*   brg          */ use_brgemm,
          /*   block_size_K */ block_size_K,
          /*   do_unpack    */ do_unpack);
    });
```
**EN:** This section drives `copy_stub`, `A_ids`, `offset` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`copy_stub`、`A_ids`、`offset`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 98-120: Runtime integration and dispatch
```cpp
    if (use_brgemm) {
      at::native::cpublas::brgemm_release();
    }
  });

  // stage 1.5: intermediate_cache1 = silu(intermediate_cache0)
  at::parallel_for(0, M * topk, 0, [&](int64_t begin, int64_t end) {
    for (int64_t m = begin; m < end; ++m) {
      silu_and_mul_stub(ic1 + m * N, ic0 + m * 2 * N, ic0 + m * 2 * N + N, N);
    }
  });

  // stage 2: intermediate_cache2 = intermediate_cache1 @ w2
  //   w2 : [E, K, N] as [E, OC, IC]
  const int64_t OC = K;  // rename K as OC
  const int64_t IC = N;  // rename N as IC
  const int64_t MB2 = MB;
  const int64_t NB2 = div_up(OC, BLOCK_N);
  scale_size_N = div_up(K, block_size_N);
  scale_size_K = div_up(N, block_size_K);
  const int64_t stride_e2 = OC * IC;
  const int64_t stride_oc = IC;
```
**EN:** This section uses `at::parallel_for`, `brgemm_release`, `silu_and_mul_stub` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`brgemm_release`、`silu_and_mul_stub`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 121-144: Runtime integration and dispatch
```cpp
  // parallel on [MB2, NB2]
  parallel_2d(MB2, NB2, [&](int64_t mb0, int64_t mb1, int64_t nb0, int64_t nb1) {
    int tid = get_thread_num();
    alignas(64) scalar_t C[BLOCK_M * BLOCK_K];

    loop_2d<at::Float8_e4m3fn>(mb0, mb1, nb0, nb1, BLOCK_N * IC, [&](int64_t mb, int64_t nb, int64_t nb_offset) {
      int64_t m_size = offsets[mb + 1] - offsets[mb];
      int64_t n_size = std::min(OC - nb * BLOCK_N, BLOCK_N);

      // A ptr from ic1 of [M * topk, N] in sorted order
      // so as to avoid copy A to tmp buffer again
      const scalar_t* __restrict__ A = ic1 + offsets[mb] * N;
      const int32_t* A_ids = sorted_ids + mb * BLOCK_M;

      // B shape [IC, n_size] in vnni format
      int32_t expert_id = expert_ids[mb];
      const at::Float8_e4m3fn* __restrict__ B = packed_w2 + expert_id * stride_e2 + nb * BLOCK_N * stride_oc;
      const float* __restrict__ Bs =
          w2s + expert_id * scale_size_N * scale_size_K + (nb / blocks_n_per_group) * scale_size_K;

      // do unpacking for the first row or a new expert
      int32_t pre_expert_id = mb == 0 ? -1 : expert_ids[mb - 1];
      bool do_unpack = (mb == mb0) || (expert_id != pre_expert_id);
```
**EN:** This section uses `parallel_2d`, `get_thread_num`, `min` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`parallel_2d`、`get_thread_num`、`min`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 145-168: Control flow and branching
```cpp
      tinygemm_kernel<scalar_t>(
          /*   A            */ A,
          /*   B            */ B,
          /*   C            */ C,
          /*   Btmp         */ B_tmp + tid * B_tmp_size_per_thread + nb_offset * BLOCK_N * IC,
          /*   Ctmp         */ C_tmp + tid * 2 * BLOCK_M * BLOCK_N,
          /*   scale        */ Bs,
          /*   M            */ m_size,
          /*   N            */ n_size,
          /*   K            */ IC,
          /*   lda          */ IC,
          /*   ldb          */ n_size,
          /*   ldc          */ BLOCK_N,
          /*   brg          */ use_brgemm,
          /*   block_size_K */ block_size_K,
          /*   do_unpack    */ do_unpack);

      // 2.b copy from C to ic2 in original order
      //   and also mul topk_weights in float32
      for (int64_t m = 0; m < m_size; ++m) {
        int32_t index = A_ids[m];
        float weight = topk_weights[index];
        copy_mul_stub(ic2 + index * K + nb * BLOCK_N, C + m * BLOCK_N, weight, n_size);
      }
```
**EN:** This section drives `copy_mul_stub` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`copy_mul_stub`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 169-192: Runtime integration and dispatch
```cpp
    });

    if (use_brgemm) {
      at::native::cpublas::brgemm_release();
    }
  });

  // stage 3: out = intermediate_cache2.sum(dim=1)
  //   from [M, topk, K] to [M, K]
  at::parallel_for(0, M, 0, [&](int64_t begin, int64_t end) {
    for (int64_t m = begin; m < end; ++m) {
      sum_stub(output + m * K, ic2 + m * topk * K, topk, K);
    }
  });
}

#define INSTANTIATE_MOE_FP8_TEMPLATE(TYPE)             \
  template void fused_experts_fp8_kernel_impl<TYPE>(   \
      TYPE* __restrict__ output,                       \
      TYPE* __restrict__ ic0,                          \
      TYPE* __restrict__ ic1,                          \
      TYPE* __restrict__ ic2,                          \
      TYPE* __restrict__ A_tmp,                        \
      TYPE* __restrict__ B_tmp,                        \
```
**EN:** This section uses `at::parallel_for`, `brgemm_release`, `sum_stub` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`brgemm_release`、`sum_stub`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 193-214: Runtime integration and dispatch
```cpp
      float* __restrict__ C_tmp,                       \
      const TYPE* __restrict__ input,                  \
      const at::Float8_e4m3fn* __restrict__ packed_w1, \
      const at::Float8_e4m3fn* __restrict__ packed_w2, \
      const float* __restrict__ w1s,                   \
      const float* __restrict__ w2s,                   \
      int64_t block_size_N,                            \
      int64_t block_size_K,                            \
      const float* __restrict__ topk_weights,          \
      const int32_t* __restrict__ sorted_ids,          \
      const int32_t* __restrict__ expert_ids,          \
      const int32_t* __restrict__ offsets,             \
      int64_t M,                                       \
      int64_t N,                                       \
      int64_t K,                                       \
      int64_t E,                                       \
      int64_t topk,                                    \
      int64_t num_tokens_post_pad)

INSTANTIATE_MOE_FP8_TEMPLATE(at::BFloat16);
INSTANTIATE_MOE_FP8_TEMPLATE(at::Half);
```
**EN:** This section uses `INSTANTIATE_MOE_FP8_TEMPLATE` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`INSTANTIATE_MOE_FP8_TEMPLATE`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 215-236: Runtime integration and dispatch
```cpp
template <typename scalar_t>
void shared_expert_fp8_kernel_impl(
    scalar_t* __restrict__ output,
    scalar_t* __restrict__ ic0,
    scalar_t* __restrict__ ic1,
    scalar_t* __restrict__ B_tmp,
    float* __restrict__ C_tmp,
    const scalar_t* __restrict__ input,
    const at::Float8_e4m3fn* __restrict__ packed_w1,
    const at::Float8_e4m3fn* __restrict__ packed_w2,
    const float* __restrict__ w1s,
    const float* __restrict__ w2s,
    int64_t block_size_N,
    int64_t block_size_K,
    const scalar_t* __restrict__ fused_experts_out,
    float routed_scaling_factor,
    int64_t M,
    int64_t N,
    int64_t K) {
  constexpr int64_t BLOCK_M = block_size_m();
  constexpr int64_t BLOCK_N = block_size_n();
```
**EN:** This section uses `shared_expert_fp8_kernel_impl`, `block_size_m`, `block_size_n` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`shared_expert_fp8_kernel_impl`、`block_size_m`、`block_size_n`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 237-257: Runtime integration and dispatch
```cpp
  // stage 1: intermediate_cache0 = hidden_states @ w1
  const int64_t MB = div_up(M, BLOCK_M);
  const int64_t NB = div_up(2 * N, BLOCK_N);
  int64_t scale_size_K = div_up(K, block_size_K);
  int64_t blocks_n_per_group = block_size_N / BLOCK_N;

  const bool use_brgemm = can_use_brgemm<at::Float8_e4m3fn>(M);
  const bool apply_scaling_factor = fused_experts_out != nullptr;

  int64_t B_tmp_size_per_thread = MAX_CACHE_BLOCK_SIZE * BLOCK_N * std::max(K, N);

  parallel_2d(MB, NB, [&](int64_t mb0, int64_t mb1, int64_t nb0, int64_t nb1) {
    int tid = get_thread_num();

    loop_2d<at::Float8_e4m3fn>(mb0, mb1, nb0, nb1, BLOCK_N * K, [&](int64_t mb, int64_t nb, int64_t nb_offset) {
      int64_t m_size = std::min(M - mb * BLOCK_M, BLOCK_M);
      int64_t n_size = std::min(2 * N - nb * BLOCK_N, BLOCK_N);

      // do unpacking for the first row
      bool do_unpack = (mb == mb0);
```
**EN:** This section uses `parallel_2d`, `div_up`, `max` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`parallel_2d`、`div_up`、`max`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 258-280: Runtime integration and dispatch
```cpp
      tinygemm_kernel<scalar_t>(
          /*   A            */ input + mb * BLOCK_M * K,
          /*   B            */ packed_w1 + nb * BLOCK_N * K,
          /*   C            */ ic0 + mb * BLOCK_M * 2 * N + nb * BLOCK_N,
          /*   Btmp         */ B_tmp + tid * B_tmp_size_per_thread + nb_offset * BLOCK_N * K,
          /*   Ctmp         */ C_tmp + tid * 2 * BLOCK_M * BLOCK_N,
          /*   scale        */ w1s + (nb / blocks_n_per_group) * scale_size_K,
          /*   M            */ m_size,
          /*   N            */ n_size,
          /*   K            */ K,
          /*   lda          */ K,
          /*   ldb          */ n_size,
          /*   ldc          */ 2 * N,
          /*   brg          */ use_brgemm,
          /*   block_size_K */ block_size_K,
          /*   do_unpack    */ do_unpack);
    });

    if (use_brgemm) {
      at::native::cpublas::brgemm_release();
    }
  });
```
**EN:** This section uses `brgemm_release` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`brgemm_release`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 281-304: Runtime integration and dispatch
```cpp
  // stage 1.5: intermediate_cache1 = silu(intermediate_cache0)
  at::parallel_for(0, M, 0, [&](int64_t begin, int64_t end) {
    for (int64_t m = begin; m < end; ++m) {
      silu_and_mul_stub(ic1 + m * N, ic0 + m * 2 * N, ic0 + m * 2 * N + N, N);
    }
  });

  // stage 2: intermediate_cache2 = intermediate_cache1 @ w2
  //   w2 : [K, N] as [OC, IC]
  const int64_t OC = K;  // rename K as OC
  const int64_t IC = N;  // rename N as IC
  const int64_t MB2 = MB;
  const int64_t NB2 = div_up(K, BLOCK_N);
  scale_size_K = div_up(N, block_size_K);

  // parallel on [MB2, NB2]
  parallel_2d(MB2, NB2, [&](int64_t mb0, int64_t mb1, int64_t nb0, int64_t nb1) {
    int tid = get_thread_num();
    alignas(64) scalar_t C[BLOCK_M * BLOCK_K];

    loop_2d<at::Float8_e4m3fn>(mb0, mb1, nb0, nb1, BLOCK_N * IC, [&](int64_t mb, int64_t nb, int64_t nb_offset) {
      int64_t m_size = std::min(M - mb * BLOCK_M, BLOCK_M);
      int64_t n_size = std::min(OC - nb * BLOCK_N, BLOCK_N);
```
**EN:** This section uses `at::parallel_for`, `parallel_2d`, `silu_and_mul_stub` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`parallel_2d`、`silu_and_mul_stub`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 305-325: Local implementation details
```cpp
      // do unpacking for the first row
      bool do_unpack = (mb == mb0);

      // 2.a gemm: C = A @ B
      tinygemm_kernel<scalar_t>(
          /*   A            */ ic1 + mb * BLOCK_M * N,
          /*   B            */ packed_w2 + nb * BLOCK_N * N,
          /*   C            */ C,
          /*   Btmp         */ B_tmp + tid * B_tmp_size_per_thread + nb_offset * BLOCK_N * IC,
          /*   Ctmp         */ C_tmp + tid * 2 * BLOCK_M * BLOCK_N,
          /*   scale        */ w2s + (nb / blocks_n_per_group) * scale_size_K,
          /*   M            */ m_size,
          /*   N            */ n_size,
          /*   K            */ IC,
          /*   lda          */ IC,
          /*   ldb          */ n_size,
          /*   ldc          */ BLOCK_N,
          /*   brg          */ use_brgemm,
          /*   block_size_K */ block_size_K,
          /*   do_unpack    */ do_unpack);
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 326-349: Runtime integration and dispatch
```cpp
      // 2.b copy from C to output and add fused_experts_out
      scalar_t* __restrict__ out = output + mb * BLOCK_M * K + nb * BLOCK_N;
      const scalar_t* __restrict__ fused_out =
          apply_scaling_factor ? fused_experts_out + mb * BLOCK_M * K + nb * BLOCK_N : nullptr;
      for (int64_t m = 0; m < m_size; ++m) {
        const scalar_t* __restrict__ fused_out_row = apply_scaling_factor ? (fused_out + m * K) : nullptr;
        add_mul_stub(out + m * K, C + m * BLOCK_N, fused_out_row, routed_scaling_factor, n_size);
      }
    });
  });

  if (use_brgemm) {
    at::native::cpublas::brgemm_release();
  }
}

#define INSTANTIATE_SHARED_EXPERT_FP8_TEMPLATE(TYPE)   \
  template void shared_expert_fp8_kernel_impl<TYPE>(   \
      TYPE* __restrict__ output,                       \
      TYPE* __restrict__ ic0,                          \
      TYPE* __restrict__ ic1,                          \
      TYPE* __restrict__ B_tmp,                        \
      float* __restrict__ C_tmp,                       \
      const TYPE* __restrict__ input,                  \
```
**EN:** This section uses `add_mul_stub`, `brgemm_release`, `fused_out` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`add_mul_stub`、`brgemm_release`、`fused_out`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 350-361: Runtime integration and dispatch
```cpp
      const at::Float8_e4m3fn* __restrict__ packed_w1, \
      const at::Float8_e4m3fn* __restrict__ packed_w2, \
      const float* __restrict__ w1s,                   \
      const float* __restrict__ w2s,                   \
      int64_t block_size_N,                            \
      int64_t block_size_K,                            \
      const TYPE* __restrict__ fused_experts_out,      \
      float routed_scaling_factor,                     \
      int64_t M,                                       \
      int64_t N,                                       \
      int64_t K)
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 362-363: Runtime integration and dispatch
```cpp
INSTANTIATE_SHARED_EXPERT_FP8_TEMPLATE(at::BFloat16);
INSTANTIATE_SHARED_EXPERT_FP8_TEMPLATE(at::Half);
```
**EN:** This section uses `INSTANTIATE_SHARED_EXPERT_FP8_TEMPLATE` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`INSTANTIATE_SHARED_EXPERT_FP8_TEMPLATE`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `common.h`, `gemm.h`, `moe.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cpu / moe_fp8.cpp
