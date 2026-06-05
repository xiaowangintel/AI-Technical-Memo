# moe.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/moe.cpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Headers and compile-time setup
```cpp
#include "moe.h"

#include "common.h"
#include "gemm.h"

namespace {

// [NOTE]: Fused MoE kernel with AMX
//
//   This file contains implementations for
//     * `moe_align_block_size`
//     * `fused_moe`
//
//   The functionality is identical to triton kernel, excepts:
//     * fuse silu_and_mul with gemm1, therefore this kernel
//       allocates 2 intermediate_caches instead of 3
//     * add `offsets` in `moe_align_block_size` which keeps track
//       of starting offset for each M block. this is for keeping
//       output of silu_and_mul in sorted order, thus load_A for
//       the 2nd gemm would be contiguous, therefore we can directly
//       load A from intermediate_cache1.
//
//  TODO:
//     1. tune BLOCK_M and BLOCK_N (BLOCK_N * K fit L2)
//     2. add prefetch for load A which is indexed access
//     3. abstract at::native::cpublas::brgemm with WoQ gemm (M = 1 & M != 1)
//
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 29-56: Runtime integration and dispatch
```cpp
template <int BLOCK_M>
int moe_align_block_size(
    int32_t* __restrict__ sorted_ids,
    int32_t* __restrict__ expert_ids,
    int32_t* __restrict__ topk_ids,
    int32_t* __restrict__ total_cnts,
    int32_t* __restrict__ cumsums,
    int32_t* __restrict__ offsets,
    int num_experts,
    int numel,
    int num_threads) {
#define T_INDEX(tt) total_cnts + (tt) * num_experts

  // accumulate count of expert ids locally
  at::parallel_for(0, numel, 0, [&](int begin, int end) {
    int tid = at::get_thread_num();
    int32_t* __restrict__ local_cnts = T_INDEX(tid + 1);

    for (int i = begin; i < end; ++i) {
      local_cnts[topk_ids[i]]++;
    }
  });

  using iVec = at::vec::Vectorized<int32_t>;
  for (int t = 0; t < num_threads; ++t) {
    at::vec::map2<int32_t>(
        [](iVec x, iVec y) { return x + y; }, T_INDEX(t + 1), T_INDEX(t + 1), T_INDEX(t), num_experts);
  }
```
**EN:** This section uses `moe_align_block_size`, `T_INDEX`, `get_thread_num` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`moe_align_block_size`、`T_INDEX`、`get_thread_num`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 57-85: Runtime integration and dispatch
```cpp

  // the last row holds sums of each experts
  int32_t* total_cnts_t_1 = T_INDEX(num_threads);

  cumsums[0] = 0;
  for (int e = 0; e < num_experts; ++e) {
    // accumulate `num_tokens_post_pad`, also as the expert offset
    cumsums[e + 1] = cumsums[e] + div_up(total_cnts_t_1[e], BLOCK_M) * BLOCK_M;

    for (int k = cumsums[e]; k < cumsums[e + 1]; k += BLOCK_M) {
      expert_ids[k / BLOCK_M] = e;
    }
  }
  int num_tokens_post_pad = cumsums[num_experts];

  at::parallel_for(0, numel, 0, [&](int begin, int end) {
    int tid = at::get_thread_num();
    // thread tid offsets in `total_cnts`
    int32_t* __restrict__ offsets = T_INDEX(tid);

    for (int i = begin; i < end; ++i) {
      int32_t expert_id = topk_ids[i];
      int32_t b_offset = cumsums[expert_id];
      int32_t t_offset = offsets[expert_id];
      sorted_ids[b_offset + t_offset] = i;
      offsets[expert_id]++;
    }
  });
```
**EN:** This section uses `at::parallel_for`, `T_INDEX`, `get_thread_num` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`T_INDEX`、`get_thread_num`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 86-114: Runtime integration and dispatch
```cpp
  // debug: the offset for thread t_1 should be identical to t_2
  int32_t* total_cnts_t_2 = T_INDEX(num_threads - 1);
  for (int e = 0; e < num_experts; ++e) {
    TORCH_CHECK(total_cnts_t_1[e] == total_cnts_t_2[e]);
  }

  // padding value for sorted_ids: numel
  auto sorted_id_size = [=](const int32_t* sorted_ids_ptr) {
    for (int d = 0; d < BLOCK_M; ++d) {
      if (sorted_ids_ptr[d] == numel) {
        return d;
      }
    }
    return BLOCK_M;
  };

  // offsets holds starting offset for each valida M blocks
  //   shape : [num_token_blocks + 1]
  offsets[0] = 0;
  const int num_token_blocks = num_tokens_post_pad / BLOCK_M;
  at::parallel_for(0, num_token_blocks, GRAIN_SIZE / BLOCK_M, [&](int begin, int end) {
    for (int mb = begin; mb < end; ++mb) {
      offsets[mb + 1] = sorted_id_size(sorted_ids + mb * BLOCK_M);
    }
  });
  // TODO: do we need to vecterize this ?
  for (int mb = 0; mb < num_token_blocks; ++mb) {
    offsets[mb + 1] += offsets[mb];
  }
```
**EN:** This section uses `at::parallel_for`, `T_INDEX`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`T_INDEX`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 115-142: Runtime integration and dispatch
```cpp
  // debug: the last value of offsets should be `numel`
  TORCH_CHECK(offsets[num_token_blocks] == numel);

  return num_tokens_post_pad;
}

//   silu :    shape          leading dimension
//  input0  [m_size, BLOCK_N]    BLOCK_N
//  input1  [m_size, BLOCK_N]    BLOCK_N
//  output  [M * topk, N]          N
template <typename scalar_t, int BLOCK_N>
inline void silu_and_mul(
    scalar_t* __restrict__ output,
    const float* __restrict__ input0,  // x: x0, x1
    const float* __restrict__ input1,  // y: y0, y1
    int64_t m_size,
    int64_t N) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;

  const fVec one = fVec(1.f);

  // no remainder
  for (int64_t m = 0; m < m_size; ++m) {
    scalar_t* __restrict__ out = output + m * N;
    const float* __restrict__ x = input0 + m * BLOCK_N;
    const float* __restrict__ y = input1 + m * BLOCK_N;
```
**EN:** This section uses `silu_and_mul`, `TORCH_CHECK`, `fVec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`silu_and_mul`、`TORCH_CHECK`、`fVec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 143-173: Types and data layout
```cpp
    for (int64_t d = 0; d < BLOCK_N; d += bVec::size()) {
      fVec x0 = fVec::loadu(x + d);
      fVec x1 = fVec::loadu(x + d + fVec::size());
      fVec y0 = fVec::loadu(y + d);
      fVec y1 = fVec::loadu(y + d + fVec::size());
      // silu
      x0 = x0 / (one + x0.neg().exp_u20());
      x1 = x1 / (one + x1.neg().exp_u20());
      // mul
      x0 = x0 * y0;
      x1 = x1 * y1;
      // convert
      bVec out_vec = convert_from_float_ext<scalar_t>(x0, x1);
      out_vec.store(out + d);
    }
  }
}

template <typename scalar_t, int BLOCK_M, int BLOCK_N>
struct tinygemm_kernel_nn2 {
  static inline void apply(
      const scalar_t* __restrict__ A,
      const scalar_t* __restrict__ B0,
      const scalar_t* __restrict__ B1,
      scalar_t* __restrict__ C,
      int64_t K,
      int64_t lda,
      int64_t ldb,
      int64_t ldc) {
    TORCH_CHECK(false, "tinygemm_kernel_nn: scalar path not implemented!");
  }
```
**EN:** This section defines `tinygemm_kernel_nn2`, `bVec::size`, `apply`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`tinygemm_kernel_nn2`、`bVec::size`、`apply`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 174-201: Types and data layout
```cpp
};

#if defined(CPU_CAPABILITY_AVX512)
template <int BLOCK_M, int BLOCK_N>
struct tinygemm_kernel_nn2<at::BFloat16, BLOCK_M, BLOCK_N> {
  static inline void apply(
      const at::BFloat16* __restrict__ A,
      const at::BFloat16* __restrict__ B0,
      const at::BFloat16* __restrict__ B1,
      at::BFloat16* __restrict__ C,
      int64_t K,
      int64_t lda,
      int64_t ldb,
      int64_t ldc) {
    constexpr int ROWS = BLOCK_M;
    constexpr int COLS = BLOCK_N / 16;

    static_assert(COLS % 2 == 0);

    // prefetch distance
    constexpr int PREFETCH_SIZE_K = 0;

    __m512bh va;
    __m512bh vb0[COLS];
    __m512bh vb1[COLS];
    __m512 vc0[ROWS * COLS];
    __m512 vc1[ROWS * COLS];
```
**EN:** This section defines `tinygemm_kernel_nn2`, `apply`, `static_assert`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`tinygemm_kernel_nn2`、`apply`、`static_assert`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 202-229: Templates, aliases, and constants
```cpp
    auto loadc = [&](auto i) {
      vc0[i] = _mm512_set1_ps(0.f);
      vc1[i] = _mm512_set1_ps(0.f);
    };
    Unroll<ROWS * COLS>{}(loadc);

    const int64_t K2 = K >> 1;
    const int64_t lda2 = lda >> 1;
    const int64_t ldb2 = ldb;  // ldb * 2 >> 1;
    const float* a_ptr = reinterpret_cast<const float*>(A);
    const float* b0_ptr = reinterpret_cast<const float*>(B0);
    const float* b1_ptr = reinterpret_cast<const float*>(B1);

    auto compute = [&](auto i, int64_t k) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;

      if constexpr (col == 0) {
        va = (__m512bh)(_mm512_set1_ps(a_ptr[row * lda2 + k]));
      }
      if constexpr (row == 0) {
        vb0[col] = (__m512bh)(_mm512_loadu_si512(b0_ptr + k * ldb2 + col * 16));
        vb1[col] = (__m512bh)(_mm512_loadu_si512(b1_ptr + k * ldb2 + col * 16));
        if constexpr (PREFETCH_SIZE_K > 0) {
          _mm_prefetch(b0_ptr + (k + PREFETCH_SIZE_K) * ldb2 + col * 16, _MM_HINT_T0);
          _mm_prefetch(b1_ptr + (k + PREFETCH_SIZE_K) * ldb2 + col * 16, _MM_HINT_T0);
        }
      }
```
**EN:** This section defines `_mm512_set1_ps`, `_mm512_loadu_si512`, `_mm_prefetch`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`_mm512_set1_ps`、`_mm512_loadu_si512`、`_mm_prefetch`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 230-258: Runtime integration and dispatch
```cpp
      vc0[i] = _mm512_dpbf16_ps(vc0[i], va, vb0[col]);
      vc1[i] = _mm512_dpbf16_ps(vc1[i], va, vb1[col]);
    };
    for (int64_t k = 0; k < K2; ++k) {
      Unroll<ROWS * COLS>{}(compute, k);
    }

    using Vec = at::vec::Vectorized<float>;
    const Vec one = Vec(1.f);
    auto storec = [&](auto i) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;
      // for COLS = 2, 4 use 512bit store
      if constexpr (col % 2 == 0) {
        Vec x0 = vc0[row * COLS + col + 0];
        Vec x1 = vc0[row * COLS + col + 1];
        Vec y0 = vc1[row * COLS + col + 0];
        Vec y1 = vc1[row * COLS + col + 1];
        // silu
        x0 = x0 / (one + x0.neg().exp_u20());
        x1 = x1 / (one + x1.neg().exp_u20());
        // mul
        x0 = x0 * y0;
        x1 = x1 * y1;

        _mm512_storeu_si512(
            reinterpret_cast<__m512i*>((C + row * ldc + col * 16)),
            (__m512i)(_mm512_cvtne2ps_pbh(__m512(x1), __m512(x0))));
      }
```
**EN:** This section uses `_mm512_dpbf16_ps`, `Vec`, `neg` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`_mm512_dpbf16_ps`、`Vec`、`neg`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 259-292: Templates, aliases, and constants
```cpp
    };
    Unroll<ROWS * COLS>{}(storec);
  }
};
#endif

#define LAUNCH_TINYGEMM_KERNEL_NN(MB_SIZE, NB_SIZE)       \
  tinygemm_kernel_nn2<scalar_t, MB_SIZE, NB_SIZE>::apply( \
      A + mb_start * lda, B0 + nb_start * 2, B1 + nb_start * 2, C + mb_start * ldc + nb_start, K, lda, ldb, ldc);

template <typename scalar_t>
void tinygemm_kernel(
    const scalar_t* __restrict__ A,
    const scalar_t* __restrict__ B0,
    const scalar_t* __restrict__ B1,
    scalar_t* __restrict__ C,
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t lda,
    int64_t ldb,
    int64_t ldc) {
  // pattern: 1-(2+2)-(8+8)
  constexpr int64_t BLOCK_M = 4;
  constexpr int64_t BLOCK_N = 32;
  const int64_t MB = div_up(M, BLOCK_M);
  const int64_t NB = div_up(N, BLOCK_N);
  for (int mb = 0; mb < MB; ++mb) {
    int64_t mb_start = mb * BLOCK_M;
    int64_t mb_size = std::min(BLOCK_M, M - mb_start);
    for (int64_t nb = 0; nb < NB; ++nb) {
      int64_t nb_start = nb * BLOCK_N;
      int64_t nb_size = std::min(BLOCK_N, N - nb_start);
```
**EN:** This section defines `tinygemm_kernel`, `LAUNCH_TINYGEMM_KERNEL_NN`, `div_up`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`tinygemm_kernel`、`LAUNCH_TINYGEMM_KERNEL_NN`、`div_up`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 293-316: Runtime integration and dispatch
```cpp
      switch (mb_size << 4 | nb_size >> 4) {
        // mb_size = 1
        case 0x12:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 32);
          break;
        // mb_size = 2
        case 0x22:
          LAUNCH_TINYGEMM_KERNEL_NN(2, 32);
          break;
        // mb_size = 3
        case 0x32:
          LAUNCH_TINYGEMM_KERNEL_NN(3, 32);
          break;
        // mb_size = 4
        case 0x42:
          LAUNCH_TINYGEMM_KERNEL_NN(4, 32);
          break;
        default:
          TORCH_CHECK(false, "Unexpected block size, ", mb_size, "x", "nb_size");
      }
    }
  }
}
```
**EN:** This section uses `LAUNCH_TINYGEMM_KERNEL_NN`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`LAUNCH_TINYGEMM_KERNEL_NN`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 317-344: Types and data layout
```cpp
template <typename scalar_t, int BLOCK_M, int BLOCK_N>
struct tinygemm_kernel_nn {
  static inline void apply(
      const scalar_t* __restrict__ A,
      const scalar_t* __restrict__ B,
      float* __restrict__ C,
      int64_t K,
      int64_t lda,
      int64_t ldb,
      int64_t ldc) {
    TORCH_CHECK(false, "tinygemm_kernel_nn: scalar path not implemented!");
  }
};

#if defined(CPU_CAPABILITY_AVX512)
template <int BLOCK_M, int BLOCK_N>
struct tinygemm_kernel_nn<at::BFloat16, BLOCK_M, BLOCK_N> {
  static inline void apply(
      const at::BFloat16* __restrict__ A,
      const at::BFloat16* __restrict__ B,
      float* __restrict__ C,
      int64_t K,
      int64_t lda,
      int64_t ldb,
      int64_t ldc) {
    constexpr int ROWS = BLOCK_M;
    constexpr int COLS = BLOCK_N / 16;
```
**EN:** This section defines `tinygemm_kernel_nn`, `apply`, `TORCH_CHECK`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`tinygemm_kernel_nn`、`apply`、`TORCH_CHECK`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 345-374: Templates, aliases, and constants
```cpp
    static_assert(COLS % 2 == 0);

    // prefetch distance
    constexpr int PREFETCH_SIZE_K = 0;

    __m512bh va;
    __m512bh vb[COLS];
    __m512 vc[ROWS * COLS];

    auto loadc = [&](auto i) { vc[i] = _mm512_set1_ps(0.f); };
    Unroll<ROWS * COLS>{}(loadc);

    const int64_t K2 = K >> 1;
    const int64_t lda2 = lda >> 1;
    const int64_t ldb2 = ldb;  // ldb * 2 >> 1;
    const float* a_ptr = reinterpret_cast<const float*>(A);
    const float* b_ptr = reinterpret_cast<const float*>(B);

    auto compute = [&](auto i, int64_t k) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;

      if constexpr (col == 0) {
        va = (__m512bh)(_mm512_set1_ps(a_ptr[row * lda2 + k]));
      }
      if constexpr (row == 0) {
        vb[col] = (__m512bh)(_mm512_loadu_si512(b_ptr + k * ldb2 + col * 16));
        if constexpr (PREFETCH_SIZE_K > 0) {
          _mm_prefetch(b_ptr + (k + PREFETCH_SIZE_K) * ldb2 + col * 16, _MM_HINT_T0);
        }
```
**EN:** This section defines `static_assert`, `_mm512_set1_ps`, `_mm512_loadu_si512`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`static_assert`、`_mm512_set1_ps`、`_mm512_loadu_si512`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 375-402: Templates, aliases, and constants
```cpp
      }
      vc[i] = _mm512_dpbf16_ps(vc[i], va, vb[col]);
    };
    for (int64_t k = 0; k < K2; ++k) {
      Unroll<ROWS * COLS>{}(compute, k);
    }

    auto storec = [&](auto i) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;
      _mm512_storeu_ps(reinterpret_cast<__m512*>(C + row * ldc + col * 16), vc[i]);
    };
    Unroll<ROWS * COLS>{}(storec);
  }
};
#endif

#define LAUNCH_TINYGEMM_KERNEL_NN2(MB_SIZE, NB_SIZE)     \
  tinygemm_kernel_nn<scalar_t, MB_SIZE, NB_SIZE>::apply( \
      A + mb_start * lda, B + nb_start * 2, C + mb_start * ldc + nb_start, K, lda, ldb, ldc);

template <typename scalar_t>
void tinygemm_kernel(
    const scalar_t* __restrict__ A,
    const scalar_t* __restrict__ B,
    float* __restrict__ C,
    int64_t M,
    int64_t N,
```
**EN:** This section defines `_mm512_dpbf16_ps`, `_mm512_storeu_ps`, `LAUNCH_TINYGEMM_KERNEL_NN2`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`_mm512_dpbf16_ps`、`_mm512_storeu_ps`、`LAUNCH_TINYGEMM_KERNEL_NN2`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 403-430: Templates, aliases, and constants
```cpp
    int64_t K,
    int64_t lda,
    int64_t ldb,
    int64_t ldc) {
  // pattern: 1-2-8
  constexpr int64_t BLOCK_M = 4;
  constexpr int64_t BLOCK_N = 32;
  const int64_t MB = div_up(M, BLOCK_M);
  const int64_t NB = div_up(N, BLOCK_N);
  for (int mb = 0; mb < MB; ++mb) {
    int64_t mb_start = mb * BLOCK_M;
    int64_t mb_size = std::min(BLOCK_M, M - mb_start);
    for (int64_t nb = 0; nb < NB; ++nb) {
      int64_t nb_start = nb * BLOCK_N;
      int64_t nb_size = std::min(BLOCK_N, N - nb_start);

      switch (mb_size << 4 | nb_size >> 4) {
        // mb_size = 1
        case 0x12:
          LAUNCH_TINYGEMM_KERNEL_NN2(1, 32);
          break;
        // mb_size = 2
        case 0x22:
          LAUNCH_TINYGEMM_KERNEL_NN2(2, 32);
          break;
        // mb_size = 3
        case 0x32:
          LAUNCH_TINYGEMM_KERNEL_NN2(3, 32);
```
**EN:** This section defines `div_up`, `min`, `LAUNCH_TINYGEMM_KERNEL_NN2`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`div_up`、`min`、`LAUNCH_TINYGEMM_KERNEL_NN2`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 431-458: Runtime integration and dispatch
```cpp
          break;
        // mb_size = 4
        case 0x42:
          LAUNCH_TINYGEMM_KERNEL_NN2(4, 32);
          break;
        default:
          TORCH_CHECK(false, "Unexpected block size, ", mb_size, "x", "nb_size");
      }
    }
  }
}

template <typename scalar_t>
void fused_experts_kernel_impl(
    scalar_t* __restrict__ output,
    scalar_t* __restrict__ ic1,
    scalar_t* __restrict__ ic2,
    scalar_t* __restrict__ A_tmp,
    float* __restrict__ C_tmp,
    const scalar_t* __restrict__ input,
    const scalar_t* __restrict__ packed_w1,
    const scalar_t* __restrict__ packed_w2,
    const float* __restrict__ topk_weights,
    const int32_t* __restrict__ sorted_ids,
    const int32_t* __restrict__ expert_ids,
    const int32_t* __restrict__ offsets,
    int64_t M,
    int64_t N,
```
**EN:** This section uses `LAUNCH_TINYGEMM_KERNEL_NN2`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`LAUNCH_TINYGEMM_KERNEL_NN2`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 459-487: Runtime integration and dispatch
```cpp
    int64_t K,
    int64_t E,
    int64_t topk,
    int64_t num_tokens_post_pad) {
  // handle 2 tiles per block
  constexpr int64_t BLOCK_M = block_size_m();
  constexpr int64_t BLOCK_N = block_size_n();

  // stage 1: intermediate_cache1 = silu(hidden_states @ w1)
  const int64_t MB = div_up(num_tokens_post_pad, BLOCK_M);
  const int64_t NB = div_up(N, BLOCK_N);

  // strides for w1: [E, 2N, K]
  TORCH_CHECK(N % BLOCK_N == 0, "Fixme when N is not multiples of ", BLOCK_N);

  const int64_t stride_e = 2 * N * K;
  const int64_t stride_n = K;

  int64_t avg_M = std::max(int64_t(1), M * topk / E);
  const bool use_brgemm = can_use_brgemm<scalar_t>(avg_M);

  // here we only parallel on half of 2N to fuse silu_and_mul with gemm
  parallel_2d(MB, NB, [&](int64_t mb0, int64_t mb1, int64_t nb0, int64_t nb1) {
    // get local pointers
    int tid = get_thread_num();
    scalar_t* __restrict__ A = A_tmp + tid * BLOCK_M * K;
    float* __restrict__ C0 = C_tmp + tid * 2 * BLOCK_M * BLOCK_N;
    float* __restrict__ C1 = C0 + BLOCK_M * BLOCK_N;
```
**EN:** This section uses `parallel_2d`, `block_size_m`, `block_size_n` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`parallel_2d`、`block_size_m`、`block_size_n`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 488-515: Runtime integration and dispatch
```cpp
    loop_2d<scalar_t>(mb0, mb1, nb0, nb1, BLOCK_N * K * 2, [&](int64_t mb, int64_t nb, int64_t nb_offset) {
      // nb_upper from top half and nb_lower from bottom half
      int64_t nb_upper = nb, nb_lower = nb + NB;
      int64_t n_size = std::min(N - nb * BLOCK_N, BLOCK_N);

      // B shape [K, n_size] in vnni format
      int32_t expert_id = expert_ids[mb];
      const scalar_t* __restrict__ B0 = packed_w1 + expert_id * stride_e + nb_upper * BLOCK_N * stride_n;
      const scalar_t* __restrict__ B1 = packed_w1 + expert_id * stride_e + nb_lower * BLOCK_N * stride_n;

      int64_t m_size = offsets[mb + 1] - offsets[mb];

      if (nb_offset == 0) {
        // 1.a load A
        const int32_t* A_ids = sorted_ids + mb * BLOCK_M;
        for (int64_t m = 0; m < m_size; ++m) {
          int32_t index = A_ids[m] / topk;
          copy_stub(A + m * K, input + index * K, K);
        }
      }

      if (use_brgemm) {
        // 1.b gemm: C0 = A @ B0
        at::native::cpublas::brgemm(
            /* M     */ m_size,
            /* N     */ n_size,
            /* K     */ K,
            /* lda   */ K,
```
**EN:** This section uses `min`, `copy_stub`, `B0` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`min`、`copy_stub`、`B0`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 516-543: Runtime integration and dispatch
```cpp
            /* ldb   */ n_size,
            /* ldc   */ BLOCK_N,
            /* add_C */ false,
            /* A     */ A,
            /* B     */ B0,
            /* C     */ C0);

        // 1.c gemm: C1 = A @ B1
        at::native::cpublas::brgemm(
            /* M     */ m_size,
            /* N     */ n_size,
            /* K     */ K,
            /* lda   */ K,
            /* ldb   */ n_size,
            /* ldc   */ BLOCK_N,
            /* add_C */ false,
            /* A     */ A,
            /* B     */ B1,
            /* C     */ C1);

        // 1.d silu and mul
        const int64_t offset = offsets[mb];
        silu_and_mul<scalar_t, BLOCK_N>(ic1 + offset * N + nb * BLOCK_N, C0, C1, m_size, N);
      } else {
        // fused 1.bcd: silu_and_mul(A @ B0, A @ B1)
        const int64_t offset = offsets[mb];
        tinygemm_kernel(
            /* A     */ A,
```
**EN:** This section uses `brgemm`, `offset` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`brgemm`、`offset`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 544-569: Runtime integration and dispatch
```cpp
            /* B0    */ B0,
            /* B1    */ B1,
            /* C     */ ic1 + offset * N + nb * BLOCK_N,
            /* M     */ m_size,
            /* N     */ n_size,
            /* K     */ K,
            /* lda   */ K,
            /* ldb   */ n_size,
            /* ldc   */ N);
      }
    });

    if (use_brgemm) {
      at::native::cpublas::brgemm_release();
    }
  });

  // stage 2: intermediate_cache2 = intermediate_cache1 @ w2
  //   w2 : [E, K, N] as [E, OC, IC]
  const int64_t OC = K;  // rename K as OC
  const int64_t IC = N;  // rename N as IC
  const int64_t MB2 = MB;
  const int64_t NB2 = div_up(OC, BLOCK_N);
  const int64_t stride_e2 = OC * IC;
  const int64_t stride_oc = IC;
```
**EN:** This section uses `brgemm_release`, `div_up`, `OC` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`brgemm_release`、`div_up`、`OC`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 570-597: Runtime integration and dispatch
```cpp
  // parallel on [MB2, NB2]
  parallel_2d(MB2, NB2, [&](int64_t mb0, int64_t mb1, int64_t nb0, int64_t nb1) {
    // get local pointers
    int tid = get_thread_num();
    // we won't be using C1 for gemm2
    float* __restrict__ C = C_tmp + tid * 2 * BLOCK_M * BLOCK_N;

    loop_2d<scalar_t>(mb0, mb1, nb0, nb1, BLOCK_N * IC, [&](int64_t mb, int64_t nb, int64_t nb_offset) {
      int64_t m_size = offsets[mb + 1] - offsets[mb];
      int64_t n_size = std::min(OC - nb * BLOCK_N, BLOCK_N);

      // A ptr from ic1 of [M * topk, N] in sorted order
      // so as to avoid copy A to tmp buffer again
      const scalar_t* __restrict__ A = ic1 + offsets[mb] * N;
      const int32_t* A_ids = sorted_ids + mb * BLOCK_M;

      // B shape [IC, n_size] in vnni format
      int32_t expert_id = expert_ids[mb];
      const scalar_t* __restrict__ B = packed_w2 + expert_id * stride_e2 + nb * BLOCK_N * stride_oc;

      // 2.a gemm: C = A @ B
      if (use_brgemm) {
        at::native::cpublas::brgemm(
            /* M     */ m_size,
            /* N     */ n_size,
            /* K     */ IC,
            /* lda   */ IC,
            /* ldb   */ n_size,
```
**EN:** This section uses `parallel_2d`, `get_thread_num`, `min` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`parallel_2d`、`get_thread_num`、`min`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 598-624: Control flow and branching
```cpp
            /* ldc   */ BLOCK_N,
            /* add_C */ false,
            /* A     */ A,
            /* B     */ B,
            /* C     */ C);
      } else {
        tinygemm_kernel(
            /* A     */ A,
            /* B     */ B,
            /* C     */ C,
            /* M     */ m_size,
            /* N     */ n_size,
            /* K     */ IC,
            /* lda   */ IC,
            /* ldb   */ n_size,
            /* ldc   */ BLOCK_N);
      }

      // 2.b copy from C to ic2 in original order
      //   and also mul topk_weights in float32
      for (int64_t m = 0; m < m_size; ++m) {
        int32_t index = A_ids[m];
        float weight = topk_weights[index];
        copy_mul_stub(ic2 + index * K + nb * BLOCK_N, C + m * BLOCK_N, weight, n_size);
      }
    });
```
**EN:** This section drives `tinygemm_kernel`, `copy_mul_stub` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`tinygemm_kernel`、`copy_mul_stub`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 625-655: Runtime integration and dispatch
```cpp
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

template <typename scalar_t>
void shared_expert_kernel_impl(
    scalar_t* __restrict__ output,
    scalar_t* __restrict__ ic1,
    float* __restrict__ C_tmp,
    scalar_t* __restrict__ input,
    const scalar_t* __restrict__ packed_w1,
    const scalar_t* __restrict__ packed_w2,
    const scalar_t* __restrict__ fused_experts_out,
    float routed_scaling_factor,
    int64_t M,
    int64_t N,
    int64_t K) {
  // handle 2 tiles per block
  constexpr int64_t BLOCK_M = block_size_m();
  constexpr int64_t BLOCK_N = block_size_n();
```
**EN:** This section uses `at::parallel_for`, `shared_expert_kernel_impl`, `brgemm_release` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`shared_expert_kernel_impl`、`brgemm_release`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 656-682: Runtime integration and dispatch
```cpp
  // stage 1: intermediate_cache1 = silu(hidden_states @ w1)
  const int64_t MB = div_up(M, BLOCK_M);
  const int64_t NB = div_up(N, BLOCK_N);

  TORCH_CHECK(N % BLOCK_N == 0, "Fixme when N is not multiples of ", BLOCK_N);
  const int64_t stride_n = K;

  const bool use_brgemm = can_use_brgemm<scalar_t>(M);

  const bool apply_scaling_factor = fused_experts_out != nullptr;

  // here we only parallel on half of 2N to fuse silu_and_mul with gemm
  parallel_2d(MB, NB, [&](int64_t mb0, int64_t mb1, int64_t nb0, int64_t nb1) {
    // get local pointers
    int tid = get_thread_num();
    float* __restrict__ C0 = C_tmp + tid * 2 * BLOCK_M * BLOCK_N;
    float* __restrict__ C1 = C0 + BLOCK_M * BLOCK_N;

    loop_2d<scalar_t>(mb0, mb1, nb0, nb1, BLOCK_N * K * 2, [&](int64_t mb, int64_t nb, int64_t nb_offset) {
      // nb_upper from top half and nb_lower from bottom half
      int64_t nb_upper = nb, nb_lower = nb + NB;
      int64_t n_size = std::min(N - nb * BLOCK_N, BLOCK_N);
      int64_t m_size = std::min(M - mb * BLOCK_M, BLOCK_M);

      // A shape [m_size, K]
      const scalar_t* A = input + mb * BLOCK_M * K;
```
**EN:** This section uses `parallel_2d`, `div_up`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`parallel_2d`、`div_up`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 683-713: Runtime integration and dispatch
```cpp
      // B shape [K, n_size] in vnni format
      const scalar_t* __restrict__ B0 = packed_w1 + nb_upper * BLOCK_N * stride_n;
      const scalar_t* __restrict__ B1 = packed_w1 + nb_lower * BLOCK_N * stride_n;

      if (use_brgemm) {
        // 1.b gemm: C0 = A @ B0
        at::native::cpublas::brgemm(
            /* M     */ m_size,
            /* N     */ n_size,
            /* K     */ K,
            /* lda   */ K,
            /* ldb   */ n_size,
            /* ldc   */ BLOCK_N,
            /* add_C */ false,
            /* A     */ A,
            /* B     */ B0,
            /* C     */ C0);

        // 1.c gemm: C1 = A @ B1
        at::native::cpublas::brgemm(
            /* M     */ m_size,
            /* N     */ n_size,
            /* K     */ K,
            /* lda   */ K,
            /* ldb   */ n_size,
            /* ldc   */ BLOCK_N,
            /* add_C */ false,
            /* A     */ A,
            /* B     */ B1,
            /* C     */ C1);
```
**EN:** This section uses `brgemm`, `B0`, `B1` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`brgemm`、`B0`、`B1`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 714-744: Runtime integration and dispatch
```cpp
        // 1.d silu and mul
        silu_and_mul<scalar_t, BLOCK_N>(ic1 + mb * BLOCK_M * N + nb * BLOCK_N, C0, C1, m_size, N);
      } else {
        // fused 1.bcd: silu_and_mul(A @ B0, A @ B1)
        tinygemm_kernel(
            /* A     */ A,
            /* B0    */ B0,
            /* B1    */ B1,
            /* C     */ ic1 + mb * BLOCK_M * N + nb * BLOCK_N,
            /* M     */ m_size,
            /* N     */ n_size,
            /* K     */ K,
            /* lda   */ K,
            /* ldb   */ n_size,
            /* ldc   */ N);
      }
    });

    if (use_brgemm) {
      at::native::cpublas::brgemm_release();
    }
  });

  // stage 2: output = intermediate_cache1 @ w2
  //   w2 : [K, N] as [OC, IC]
  const int64_t OC = K;  // rename K as OC
  const int64_t IC = N;  // rename N as IC
  const int64_t MB2 = MB;
  const int64_t NB2 = div_up(OC, BLOCK_N);
  const int64_t stride_oc = IC;
```
**EN:** This section uses `tinygemm_kernel`, `brgemm_release`, `div_up` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`tinygemm_kernel`、`brgemm_release`、`div_up`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 745-772: Runtime integration and dispatch
```cpp
  // parallel on [MB2, NB2]
  parallel_2d(MB2, NB2, [&](int64_t mb0, int64_t mb1, int64_t nb0, int64_t nb1) {
    // get local pointers
    int tid = get_thread_num();
    // we won't be using C1 for gemm2
    float* __restrict__ C = C_tmp + tid * 2 * BLOCK_M * BLOCK_N;

    loop_2d<scalar_t>(mb0, mb1, nb0, nb1, BLOCK_N * IC, [&](int64_t mb, int64_t nb, int64_t nb_offset) {
      int64_t m_size = std::min(M - mb * BLOCK_M, BLOCK_M);
      int64_t n_size = std::min(OC - nb * BLOCK_N, BLOCK_N);

      // A shape [m_size, IC]
      const scalar_t* __restrict__ A = ic1 + mb * BLOCK_M * N;

      // B shape [IC, n_size] in vnni format
      const scalar_t* __restrict__ B = packed_w2 + nb * BLOCK_N * stride_oc;

      // 2.a gemm: C = A @ B
      if (use_brgemm) {
        at::native::cpublas::brgemm(
            /* M     */ m_size,
            /* N     */ n_size,
            /* K     */ IC,
            /* lda   */ IC,
            /* ldb   */ n_size,
            /* ldc   */ BLOCK_N,
            /* add_C */ false,
            /* A     */ A,
```
**EN:** This section uses `parallel_2d`, `get_thread_num`, `min` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`parallel_2d`、`get_thread_num`、`min`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 773-800: Runtime integration and dispatch
```cpp
            /* B     */ B,
            /* C     */ C);
      } else {
        tinygemm_kernel(
            /* A     */ A,
            /* B     */ B,
            /* C     */ C,
            /* M     */ m_size,
            /* N     */ n_size,
            /* K     */ IC,
            /* lda   */ IC,
            /* ldb   */ n_size,
            /* ldc   */ BLOCK_N);
      }

      // 2.b copy from C to output and add fused_experts_out
      scalar_t* __restrict__ out = output + mb * BLOCK_M * K + nb * BLOCK_N;
      const scalar_t* __restrict__ fused_out =
          apply_scaling_factor ? fused_experts_out + mb * BLOCK_M * K + nb * BLOCK_N : nullptr;
      for (int64_t m = 0; m < m_size; ++m) {
        const scalar_t* __restrict__ fused_out_row = apply_scaling_factor ? (fused_out + m * K) : nullptr;
        add_mul_stub(out + m * K, C + m * BLOCK_N, fused_out_row, routed_scaling_factor, n_size);
      }
    });

    if (use_brgemm) {
      at::native::cpublas::brgemm_release();
    }
```
**EN:** This section uses `tinygemm_kernel`, `add_mul_stub`, `brgemm_release` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`tinygemm_kernel`、`add_mul_stub`、`brgemm_release`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 801-824: Runtime integration and dispatch
```cpp
  });
}

}  // anonymous namespace

// common checks
static inline void check_moe_scales(
    bool use_int8_w8a8,
    bool use_fp8_w8a16,
    const std::optional<at::Tensor>& w1_scale,
    const std::optional<at::Tensor>& w2_scale,
    const std::optional<std::vector<int64_t>> block_size) {
  if (use_int8_w8a8) {
    TORCH_CHECK(w1_scale.has_value(), "missing w1_scale for int8 w8a8.");
    TORCH_CHECK(w2_scale.has_value(), "missing w2_scale for int8 w8a8.");
  }
  if (use_fp8_w8a16) {
    TORCH_CHECK(w1_scale.has_value(), "missing w1_scale for fp8 w8a16.");
    TORCH_CHECK(w2_scale.has_value(), "missing w2_scale for fp8 w8a16.");
    TORCH_CHECK(block_size.has_value(), "missing block_size for fp8 w8a16.");
    TORCH_CHECK(block_size.value().size() == 2, "expect block_size.size() to be 2.");
  }
}
```
**EN:** This section uses `check_moe_scales`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`check_moe_scales`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 825-852: Runtime integration and dispatch
```cpp
#define CHECK_MOE_SCALES_FP8(DIM0, DIM1)                      \
  auto w1s = w1_scale.value();                                \
  auto w2s = w2_scale.value();                                \
  auto block_size_val = block_size.value();                   \
  int64_t block_size_N = block_size_val[0];                   \
  int64_t block_size_K = block_size_val[1];                   \
  TORCH_CHECK(w1s.size(DIM0) == div_up(2 * N, block_size_N)); \
  TORCH_CHECK(w1s.size(DIM1) == div_up(K, block_size_K));     \
  TORCH_CHECK(w2s.size(DIM0) == div_up(K, block_size_N));     \
  TORCH_CHECK(w2s.size(DIM1) == div_up(N, block_size_K))

// hidden_states: [M, K]
// w1: [E, 2N, K]
// w2: [E, K, N]
// topk_weights: [M, topk]
// topk_ids: [M, topk] (int32_t)
//

at::Tensor fused_experts_cpu(
    at::Tensor& hidden_states,
    at::Tensor& w1,
    at::Tensor& w2,
    at::Tensor& topk_weights,
    at::Tensor& topk_ids,
    bool inplace,
    int64_t moe_comp_method,
    const std::optional<at::Tensor>& w1_scale,
    const std::optional<at::Tensor>& w2_scale,
```
**EN:** This section uses `CHECK_MOE_SCALES_FP8`, `value`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_MOE_SCALES_FP8`、`value`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 853-880: Runtime integration and dispatch
```cpp
    const std::optional<at::Tensor>& w1_zero,
    const std::optional<at::Tensor>& w2_zero,
    const std::optional<std::vector<int64_t>> block_size,
    bool is_vnni) {
  auto packed_w1 = is_vnni ? w1 : convert_weight_packed(w1);
  auto packed_w2 = is_vnni ? w2 : convert_weight_packed(w2);

  constexpr int64_t BLOCK_M = block_size_m();
  constexpr int64_t BLOCK_N = block_size_n();

  const auto st = hidden_states.scalar_type();
  CHECK_INPUT(hidden_states);
  CHECK_INPUT(w1);
  CHECK_INPUT(w2);
  CHECK_EQ(topk_weights.sizes(), topk_ids.sizes());
  CHECK_DIM(2, hidden_states);
  if (moe_comp_method == CPUQuantMethod::INT4_W4A8 && is_vnni) {
    CHECK_DIM(4, w1);
    CHECK_DIM(4, w2);
  } else {
    CHECK_DIM(3, w1);
    CHECK_DIM(3, w2);
  }
  CHECK_DIM(2, topk_weights);
  CHECK_DIM(2, topk_ids);

  CHECK_EQ(topk_ids.scalar_type(), at::kInt);
```
**EN:** This section uses `convert_weight_packed`, `block_size_m`, `block_size_n` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`convert_weight_packed`、`block_size_m`、`block_size_n`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 881-912: Runtime integration and dispatch
```cpp
  // TODO: support topk_weights to be bf16 or fp16 in the kernel.
  // The topk_weights of llama4 is computed via Llama4MoE:custom_routing_function and is bf16/fp16
  // while the kernel currently only supports it to be float32
  auto topk_weights_ = topk_weights.to(at::kFloat);
  CHECK_EQ(topk_weights_.scalar_type(), at::kFloat);

  int64_t M = hidden_states.size(0);
  int64_t K = hidden_states.size(1);
  int64_t N = moe_comp_method == CPUQuantMethod::INT4_W4A8 ? w1_scale.value().size(1) * w1_scale.value().size(3) / 2
                                                           : w1.size(1) / 2;
  int64_t E = w1.size(0);
  int64_t topk = topk_weights_.size(1);

  // we use int32_t compensation for int8 w8a8
  int64_t packed_K = get_row_size(K, moe_comp_method == CPUQuantMethod::INT8_W8A8);
  int64_t packed_N = get_row_size(N, moe_comp_method == CPUQuantMethod::INT8_W8A8);

  // check weight shapes
  CHECK_EQ(w2.size(0), E);
  if (!(moe_comp_method == CPUQuantMethod::INT4_W4A8)) {
    CHECK_EQ(w2.size(1), K);
    CHECK_EQ(packed_w1.size(2), packed_K / (moe_comp_method == CPUQuantMethod::INT4_W4A8 ? 2 : 1));
    CHECK_EQ(packed_w2.size(2), packed_N / (moe_comp_method == CPUQuantMethod::INT4_W4A8 ? 2 : 1));
  }
  // check scales
  check_moe_scales(
      moe_comp_method == CPUQuantMethod::INT8_W8A8,
      moe_comp_method == CPUQuantMethod::FP8_W8A16,
      w1_scale,
      w2_scale,
      block_size);
```
**EN:** This section uses `to`, `CHECK_EQ`, `get_row_size` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`to`、`CHECK_EQ`、`get_row_size`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 913-934: Runtime integration and dispatch
```cpp
  at::Tensor out_hidden_states = inplace ? hidden_states : at::empty_like(hidden_states);

  // NB: worst case is each expert holds a block with remainder of 1
  //   1. sorted_ids : [M * topk + E * (BLOCK_M - 1)]
  //   2. expert_ids : [max_num_blocks]
  //   3. total_cnts : [T + 1, E]
  //   4. cumsums    : [E + 1]
  //   5. offsets    : [max_num_blocks + 1]
  //
  int num_threads = at::get_num_threads();
  int64_t max_num_tokens_padded = M * topk + E * (BLOCK_M - 1);
  int64_t max_num_blocks = div_up(max_num_tokens_padded, BLOCK_M);
  auto buffer = at::empty(
      {max_num_tokens_padded + max_num_blocks + (num_threads + 1) * E + (E + 1) + (max_num_blocks + 1)},
      topk_ids.options());

  int32_t* __restrict__ sorted_ids = buffer.data_ptr<int32_t>();
  int32_t* __restrict__ expert_ids = sorted_ids + max_num_tokens_padded;
  int32_t* __restrict__ total_cnts = expert_ids + max_num_blocks;
  int32_t* __restrict__ cumsums = total_cnts + (num_threads + 1) * E;
  int32_t* __restrict__ offsets = cumsums + (E + 1);
```
**EN:** This section uses `empty_like`, `get_num_threads`, `div_up` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`empty_like`、`get_num_threads`、`div_up`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 935-962: Runtime integration and dispatch
```cpp
  // init sorted_ids with `numel` as the padding number
  // init expert_ids with `num_experts`
  int64_t numel = M * topk;
  at::parallel_for(0, max_num_blocks, GRAIN_SIZE / BLOCK_M, [&](int64_t begin, int64_t end) {
    int64_t m_start = begin * BLOCK_M;
    int64_t m_size = std::min((end - begin) * BLOCK_M, max_num_tokens_padded - m_start);
    fill_stub(sorted_ids + m_start, (int32_t)numel, m_size);
    fill_stub(expert_ids + begin, (int32_t)E, end - begin);
  });
  // zero total_cnts and cumsums
  at::parallel_for(0, (num_threads + 1) * E + (E + 1), GRAIN_SIZE, [&](int64_t begin, int64_t end) {
    fill_stub(total_cnts + begin, 0, end - begin);
  });

  // align experts index
  int64_t num_tokens_post_pad = moe_align_block_size<BLOCK_M>(
      sorted_ids, expert_ids, topk_ids.data_ptr<int32_t>(), total_cnts, cumsums, offsets, E, numel, num_threads);

  // unlike triton kernel, we fuse silu with gemm1 so only need 2 intermediate_caches:
  //   1. intermediate_cache1 : [M * topk, N]
  //   2. intermediate_cache2 : [M * topk, K]
  //   3. A_tmp : [T, BLOCK_M * K]
  //   4. C_tmp : [T, 2 * BLOCK_M * BLOCK_N]
  //
  // for int8 w8a8:
  //   5. Aq_tmp : [M, K] or [M * topk, N]
  //   6. As_tmp : [M * topk]
  //
```
**EN:** This section uses `at::parallel_for`, `min`, `fill_stub` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`min`、`fill_stub`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 963-988: Runtime integration and dispatch
```cpp
  // for fp8 w8a16:
  //   7. intermediate_cache0 : [M * topk, 2N]
  //   8. B_tmp : [T, MAX_CACHE_BLOCK_SIZE, BLOCK_N, std::max(K, N)]
  //
  int64_t buffer_size_nbytes =
      M * topk * N * 2 + M * topk * K * 2 +
      num_threads * BLOCK_M * K *
          (moe_comp_method == CPUQuantMethod::INT8_W8A8 | moe_comp_method == CPUQuantMethod::INT4_W4A8 ? 1 : 2) +
      num_threads * 2 * BLOCK_M * BLOCK_N * sizeof(float);

  if (moe_comp_method == CPUQuantMethod::INT8_W8A8) {
    buffer_size_nbytes += std::max(M * K, M * topk * N) + M * topk * sizeof(float);
  }
  if (moe_comp_method == CPUQuantMethod::FP8_W8A16) {
    buffer_size_nbytes += M * topk * 2 * N * 2 + num_threads * MAX_CACHE_BLOCK_SIZE * BLOCK_N * std::max(K, N) * 2;
  }
  if (moe_comp_method == CPUQuantMethod::INT4_W4A8) {
    buffer_size_nbytes += M * topk * 2 * N * 2 + std::max(M * K, M * topk * N) + M * topk * sizeof(float) +
                          num_threads * 2 * get_4bit_block_k_size(K / w1_scale.value().size(2)) * BLOCK_N;
  }
  auto buffer2 = at::empty({buffer_size_nbytes}, hidden_states.options().dtype(at::kChar));

  AT_DISPATCH_REDUCED_FLOATING_TYPES(st, "fused_experts_kernel_impl", [&] {
    scalar_t* __restrict__ intermediate_cache1 = (scalar_t*)((void*)(buffer2.data_ptr<int8_t>()));
    scalar_t* __restrict__ intermediate_cache2 = intermediate_cache1 + M * topk * N;
```
**EN:** This section uses `max`, `options` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`max`、`options`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 989-1016: Runtime integration and dispatch
```cpp
    if (moe_comp_method == CPUQuantMethod::INT8_W8A8) {
      uint8_t* __restrict__ A_tmp = (uint8_t*)((void*)(intermediate_cache2 + M * topk * K));
      float* __restrict__ C_tmp = (float*)((void*)(A_tmp + num_threads * BLOCK_M * K));
      uint8_t* __restrict__ Aq_tmp = (uint8_t*)((void*)(C_tmp + num_threads * 2 * BLOCK_M * BLOCK_N));
      float* __restrict__ As_tmp = (float*)((void*)(Aq_tmp + std::max(M * K, M * topk * N)));

      auto w1s = w1_scale.value();
      auto w2s = w2_scale.value();
      TORCH_CHECK(w1s.numel() == E * 2 * N);
      TORCH_CHECK(w2s.numel() == E * K);

      fused_experts_int8_kernel_impl<scalar_t>(
          out_hidden_states.data_ptr<scalar_t>(),
          intermediate_cache1,
          intermediate_cache2,
          A_tmp,
          C_tmp,
          Aq_tmp,
          As_tmp,
          hidden_states.data_ptr<scalar_t>(),
          packed_w1.data_ptr<int8_t>(),
          packed_w2.data_ptr<int8_t>(),
          w1s.data_ptr<float>(),
          w2s.data_ptr<float>(),
          topk_weights_.data_ptr<float>(),
          sorted_ids,
          expert_ids,
          offsets,
```
**EN:** This section uses `max`, `value`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`max`、`value`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1017-1044: Runtime integration and dispatch
```cpp
          M,
          N,
          K,
          E,
          topk,
          num_tokens_post_pad);
    } else if (moe_comp_method == CPUQuantMethod::FP8_W8A16) {
      // here we just ignore C_tmp as it is not used
      scalar_t* __restrict__ A_tmp = (scalar_t*)((void*)(intermediate_cache2 + M * topk * K));
      float* __restrict__ C_tmp = (float*)((void*)(A_tmp + num_threads * BLOCK_M * K));
      scalar_t* __restrict__ intermediate_cache0 = (scalar_t*)((void*)(C_tmp + num_threads * 2 * BLOCK_M * BLOCK_N));
      scalar_t* __restrict__ B_tmp = (scalar_t*)((void*)(intermediate_cache0 + M * topk * 2 * N));

      CHECK_MOE_SCALES_FP8(1, 2);
      fused_experts_fp8_kernel_impl(
          out_hidden_states.data_ptr<scalar_t>(),
          intermediate_cache0,
          intermediate_cache1,
          intermediate_cache2,
          A_tmp,
          B_tmp,
          C_tmp,
          hidden_states.data_ptr<scalar_t>(),
          packed_w1.data_ptr<at::Float8_e4m3fn>(),
          packed_w2.data_ptr<at::Float8_e4m3fn>(),
          w1s.data_ptr<float>(),
          w2s.data_ptr<float>(),
          block_size_N,
```
**EN:** This section uses `CHECK_MOE_SCALES_FP8` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_MOE_SCALES_FP8`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1045-1072: Control flow and branching
```cpp
          block_size_K,
          topk_weights_.data_ptr<float>(),
          sorted_ids,
          expert_ids,
          offsets,
          M,
          N,
          K,
          E,
          topk,
          num_tokens_post_pad);
    } else if (moe_comp_method == CPUQuantMethod::INT4_W4A8) {
      uint8_t* __restrict__ A_tmp = (uint8_t*)((void*)(intermediate_cache2 + M * topk * K));
      float* __restrict__ C_tmp = (float*)((void*)(A_tmp + num_threads * BLOCK_M * K));
      scalar_t* __restrict__ intermediate_cache0 = (scalar_t*)((void*)(C_tmp + num_threads * 2 * BLOCK_M * BLOCK_N));
      uint8_t* __restrict__ Aq_tmp = (uint8_t*)((void*)(intermediate_cache0 + M * topk * 2 * N));
      float* __restrict__ As_tmp = (float*)((void*)(Aq_tmp + std::max(M * K, M * topk * N)));
      int8_t* __restrict__ dqB_tmp = (int8_t*)((void*)(As_tmp + M * topk));

      // weight + compensation shape = [Nc, Kc, block_n * block_k / 2 + block_n*sizeof(int32_t)]
      // scales/qzeros shape = [E, Nc, G, block_n]
      int64_t num_groups = w1_scale.value().size(2);
      const int group_size = K / num_groups;
      // TODO: check scales and zeros
      fused_experts_int4_w4a8_kernel_impl<scalar_t>(
          out_hidden_states.data_ptr<scalar_t>(),
          intermediate_cache0,
          intermediate_cache1,
```
**EN:** This section drives `max`, `value`, `group_size` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`max`、`value`、`group_size`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1073-1101: Local implementation details
```cpp
          intermediate_cache2,
          A_tmp,
          Aq_tmp,
          As_tmp,
          nullptr,
          C_tmp,
          dqB_tmp,
          hidden_states.data_ptr<scalar_t>(),
          packed_w1.data_ptr<uint8_t>(),
          packed_w2.data_ptr<uint8_t>(),
          w1_zero.value().data_ptr<int8_t>(),
          w2_zero.value().data_ptr<int8_t>(),
          w1_scale.value().data_ptr<float>(),
          w2_scale.value().data_ptr<float>(),
          group_size,
          topk_weights.data_ptr<float>(),
          sorted_ids,
          expert_ids,
          offsets,
          M,
          N,
          K,
          E,
          topk,
          num_tokens_post_pad);
    } else {
      scalar_t* __restrict__ A_tmp = intermediate_cache2 + M * topk * K;
      float* __restrict__ C_tmp = (float*)((void*)(A_tmp + num_threads * BLOCK_M * K));
```
**EN:** This section fills in the local implementation details around `value`, completing the behavior required by the file.
**CN:** 本段补充了`value`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 1102-1125: Local implementation details
```cpp
      fused_experts_kernel_impl<scalar_t>(
          out_hidden_states.data_ptr<scalar_t>(),
          intermediate_cache1,
          intermediate_cache2,
          A_tmp,
          C_tmp,
          hidden_states.data_ptr<scalar_t>(),
          packed_w1.data_ptr<scalar_t>(),
          packed_w2.data_ptr<scalar_t>(),
          topk_weights_.data_ptr<float>(),
          sorted_ids,
          expert_ids,
          offsets,
          M,
          N,
          K,
          E,
          topk,
          num_tokens_post_pad);
    }
  });
  return out_hidden_states;
}
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 1126-1150: Runtime integration and dispatch
```cpp
// shared expert kernel
//
// hidden_states: [M, K]
// w1: [2N, K]
// w2: [K, N]
// fused_experts_out
at::Tensor shared_expert_cpu(
    at::Tensor& hidden_states,
    at::Tensor& w1,
    at::Tensor& w2,
    const std::optional<at::Tensor>& fused_experts_out,
    const std::optional<double> routed_scaling_factor,
    bool inplace,
    bool use_int8_w8a8,
    bool use_fp8_w8a16,
    const std::optional<at::Tensor>& w1_scale,
    const std::optional<at::Tensor>& w2_scale,
    const std::optional<std::vector<int64_t>> block_size,
    bool is_vnni) {
  auto packed_w1 = is_vnni ? w1 : convert_weight_packed(w1);
  auto packed_w2 = is_vnni ? w2 : convert_weight_packed(w2);

  constexpr int64_t BLOCK_M = block_size_m();
  constexpr int64_t BLOCK_N = block_size_n();
```
**EN:** This section uses `shared_expert_cpu`, `convert_weight_packed`, `block_size_m` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`shared_expert_cpu`、`convert_weight_packed`、`block_size_m`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1151-1176: Runtime integration and dispatch
```cpp
  double routed_scaling_factor_value = 0;
  if (routed_scaling_factor.has_value()) {
    TORCH_CHECK(fused_experts_out.has_value(), "shared_expert_cpu: expect fused_experts_out.");
    const auto fused_experts_out_tensor = fused_experts_out.value();
    routed_scaling_factor_value = routed_scaling_factor.value();
    CHECK_INPUT(fused_experts_out_tensor);
    CHECK_EQ(hidden_states.sizes(), fused_experts_out_tensor.sizes());
  }

  const auto st = hidden_states.scalar_type();
  CHECK_INPUT(hidden_states);
  CHECK_INPUT(w1);
  CHECK_INPUT(w2);
  CHECK_DIM(2, hidden_states);
  CHECK_DIM(2, w1);
  CHECK_DIM(2, w2);
  CHECK_EQ(hidden_states.scalar_type(), st);

  int64_t M = hidden_states.size(0);
  int64_t K = hidden_states.size(1);
  int64_t N = w1.size(0) / 2;

  // we use int32_t compensation for int8 w8a8
  int64_t packed_K = get_row_size(K, use_int8_w8a8);
  int64_t packed_N = get_row_size(N, use_int8_w8a8);
```
**EN:** This section uses `TORCH_CHECK`, `value`, `CHECK_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`、`value`、`CHECK_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1177-1204: Runtime integration and dispatch
```cpp
  // check weight shapes
  CHECK_EQ(w2.size(0), K);
  CHECK_EQ(packed_w1.size(1), packed_K);
  CHECK_EQ(packed_w2.size(1), packed_N);

  // check scales
  check_moe_scales(use_int8_w8a8, use_fp8_w8a16, w1_scale, w2_scale, block_size);

  at::Tensor out_hidden_states = inplace ? hidden_states : at::empty_like(hidden_states);

  // unlike triton kernel, we fuse silu with gemm1 so only need 2 intermediate_caches:
  //   1. intermediate_cache1 : [M, N]
  //   2. C_tmp : [T, 2 * BLOCK_M * BLOCK_N]
  //
  // for int8 w8a8:
  //   3. Aq_tmp : [M, K] or [M, N]
  //   4. As_tmp : [M]
  //
  // for fp8 w8a16:
  //   5. intermediate_cache0 : [M, 2N]
  //   6. B_tmp: [T, MAX_CACHE_BLOCK_SIZE, BLOCK_M, max(K, N)]
  //
  int num_threads = at::get_num_threads();
  int64_t buffer_size_nbytes = M * N * 2 + num_threads * 2 * BLOCK_M * BLOCK_N * sizeof(float);

  if (use_int8_w8a8) {
    buffer_size_nbytes += std::max(M * K, M * N) + M * sizeof(float);
  }
```
**EN:** This section uses `CHECK_EQ`, `check_moe_scales`, `empty_like` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_EQ`、`check_moe_scales`、`empty_like`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1205-1232: Runtime integration and dispatch
```cpp
  if (use_fp8_w8a16) {
    buffer_size_nbytes += M * 2 * N * 2 + num_threads * MAX_CACHE_BLOCK_SIZE * BLOCK_M * std::max(K, N) * 2;
  }

  auto buffer = at::empty({buffer_size_nbytes}, hidden_states.options().dtype(at::kChar));
  AT_DISPATCH_REDUCED_FLOATING_TYPES(st, "share_experts_kernel_impl", [&] {
    scalar_t* __restrict__ intermediate_cache1 = (scalar_t*)((void*)(buffer.data_ptr<int8_t>()));
    float* __restrict__ C_tmp = (float*)((void*)(intermediate_cache1 + M * N));

    if (use_int8_w8a8) {
      uint8_t* __restrict__ Aq_tmp = (uint8_t*)((void*)(C_tmp + num_threads * 2 * BLOCK_M * BLOCK_N));
      float* __restrict__ As_tmp = (float*)((void*)(Aq_tmp + std::max(M * K, M * N)));

      auto w1s = w1_scale.value();
      auto w2s = w2_scale.value();
      TORCH_CHECK(w1s.numel() == 2 * N);
      TORCH_CHECK(w2s.numel() == K);

      shared_expert_int8_kernel_impl<scalar_t>(
          out_hidden_states.data_ptr<scalar_t>(),
          intermediate_cache1,
          C_tmp,
          Aq_tmp,
          As_tmp,
          hidden_states.data_ptr<scalar_t>(),
          packed_w1.data_ptr<int8_t>(),
          packed_w2.data_ptr<int8_t>(),
          w1s.data_ptr<float>(),
```
**EN:** This section uses `options`, `max`, `value` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`options`、`max`、`value`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1233-1260: Runtime integration and dispatch
```cpp
          w2s.data_ptr<float>(),
          conditional_data_ptr<scalar_t>(fused_experts_out),
          routed_scaling_factor_value,
          M,
          N,
          K);
    } else if (use_fp8_w8a16) {
      scalar_t* __restrict__ intermediate_cache0 = (scalar_t*)((void*)(C_tmp + num_threads * 2 * BLOCK_M * BLOCK_N));
      scalar_t* __restrict__ B_tmp = (scalar_t*)((void*)(intermediate_cache0 + M * 2 * N));

      CHECK_MOE_SCALES_FP8(0, 1);
      shared_expert_fp8_kernel_impl<scalar_t>(
          out_hidden_states.data_ptr<scalar_t>(),
          intermediate_cache0,
          intermediate_cache1,
          B_tmp,
          C_tmp,
          hidden_states.data_ptr<scalar_t>(),
          packed_w1.data_ptr<at::Float8_e4m3fn>(),
          packed_w2.data_ptr<at::Float8_e4m3fn>(),
          w1s.data_ptr<float>(),
          w2s.data_ptr<float>(),
          block_size_N,
          block_size_K,
          conditional_data_ptr<scalar_t>(fused_experts_out),
          routed_scaling_factor_value,
          M,
          N,
```
**EN:** This section uses `CHECK_MOE_SCALES_FP8` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_MOE_SCALES_FP8`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1261-1278: Local implementation details
```cpp
          K);
    } else {
      shared_expert_kernel_impl<scalar_t>(
          out_hidden_states.data_ptr<scalar_t>(),
          intermediate_cache1,
          C_tmp,
          hidden_states.data_ptr<scalar_t>(),
          packed_w1.data_ptr<scalar_t>(),
          packed_w2.data_ptr<scalar_t>(),
          conditional_data_ptr<scalar_t>(fused_experts_out),
          routed_scaling_factor_value,
          M,
          N,
          K);
    }
  });
  return out_hidden_states;
}
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `moe.h`, `common.h`, `gemm.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cpu / moe.cpp
