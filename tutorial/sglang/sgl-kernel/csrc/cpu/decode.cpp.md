# decode.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/decode.cpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: Namespace and shared declarations
```cpp
#include "common.h"
#include "gemm.h"
#include "vec.h"

namespace {

// [NOTE] TODO list for this kernel:
//   1. tune the value for BLOCK_N
//   2. planning for {batches, num_heads, num_kv_splits}
//      and use actual num_kv_splits for small seq length
//   3. try fast impl of `.tanh()`
//   4. provide amx kernel for index_gemm_kernel_nn when M = 16
//

#if defined(CPU_CAPABILITY_AVX512)
// key: from [N, 32] to [32/2, N, 2]
// val: from [N, 32] to [N/2, 32, 2]
template <typename scalar_t, typename index_t>
inline void pack_vnni_Nx32(
    scalar_t* __restrict__ dst0,
    scalar_t* __restrict__ dst1,
    const scalar_t* __restrict__ src,
    const index_t* __restrict__ ind,
    int N,
    int ld_src,
    int ld_dst0,
    int ld_dst1,
    bool convert_v) {
  __m512i vinputs[16];
  int n = 0;
  for (; n < N; ++n) {
    vinputs[n] = _mm512_loadu_si512(src + ind[n] * ld_src);
  }
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 34-59: Control flow and branching
```cpp
  // padding with zero to avoid uninitialized vectors
  for (; n < 16; ++n) {
    vinputs[n] = _mm512_set1_epi32(0);
  }

  // pack value, skip 64 elems for deepseek
  // handle 2 vectors at a time from [2, 32] to [32, 2]
  if (convert_v) {
    for (int n = 0; n < 16; n += 2) {
      __m512i d0, d1;
      std::tie(d0, d1) = transpose_2x32_16bit(vinputs[n], vinputs[n + 1]);
      _mm512_storeu_si512(dst1 + (n >> 1) * ld_dst1 * 2, d0);
      _mm512_storeu_si512(dst1 + (n >> 1) * ld_dst1 * 2 + 32, d1);
    }
  }

  // pack key
  transpose_16x16_32bit(vinputs);

  const __mmask16 vmask = (1 << N) - 1;
  for (int k = 0; k < 16; ++k) {
    _mm512_mask_storeu_epi32(dst0 + k * ld_dst0 * 2, vmask, vinputs[k]);
  }
}
#endif
```
**EN:** This section drives `_mm512_set1_epi32`, `tie`, `_mm512_storeu_si512` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`_mm512_set1_epi32`、`tie`、`_mm512_storeu_si512`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 60-84: Templates, aliases, and constants
```cpp
// [NOTE]: MLA vnni format conversion
//
//  here we apply same strategy as `FlashMLA`:
//  each kv_cache is loaded once and packed twice (L2 cache hit)
//
//  * for   key: from [N, K/2, 2] to [K/2, N, 2]
//  * for value: from [N/2, 2, Kv] to [N/2, Kv, 2]
//
template <typename scalar_t, typename index_t>
void pack_vnni(
    scalar_t* __restrict__ dst0,
    scalar_t* __restrict__ dst1,
    const scalar_t* __restrict__ src,
    const index_t* __restrict__ ind,
    int N,
    int K,
    int Kv,
    int ld_src,
    int ld_dst0,
    int ld_dst1) {
#if defined(CPU_CAPABILITY_AVX512)
  const int NB = div_up(N, 16);
  const int KB = K / 32;    // no remainder
  const int KBv = Kv / 32;  // no remainder
```
**EN:** This section defines `pack_vnni`, `defined`, `NB`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`pack_vnni`、`defined`、`NB`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 85-109: Control flow and branching
```cpp
  for (int nb = 0; nb < NB; ++nb) {
    for (int kb = 0; kb < KB; ++kb) {
      // handle 16x512bits each block
      int nb_size = std::min(N - nb * 16, 16);
      pack_vnni_Nx32<scalar_t, index_t>(
          /*    dst0 */ dst0 + ((kb * 32) >> 1) * ld_dst0 * 2 + nb * 16 * 2,
          /*    dst1 */ dst1 + ((nb * 16) >> 1) * ld_dst1 * 2 + kb * 32 * 2,
          /*     src */ src + kb * 32,
          /*     ind */ ind + nb * 16,
          /*       N */ nb_size,
          /*  ld_src */ ld_src,
          /* ld_dst0 */ ld_dst0,
          /* ld_dst1 */ ld_dst1,
          /*   cvt_v */ kb < KBv);
    }
  }
#else
  for (int n = 0; n < N; ++n) {
    index_t index = ind[n];
    for (int k = 0; k < K / 2; ++k) {
      for (int d = 0; d < 2; ++d) {
        dst0[k * ld_dst0 * 2 + n * 2 + d] = src[index * ld_src + k * 2 + d];
      }
    }
  }
```
**EN:** This section drives `min` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`min`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 110-138: Runtime integration and dispatch
```cpp
  // from [N/2, 2, K] to [N/2, K, 2]
  for (int n = 0; n < (N >> 1) * 2; n += 2) {
    index_t index0 = ind[n + 0];
    index_t index1 = ind[n + 1];
    for (int k = 0; k < Kv; ++k) {
      dst1[(n >> 1) * ld_dst1 * 2 + k * 2 + 0] = src[index0 * ld_src + k];
      dst1[(n >> 1) * ld_dst1 * 2 + k * 2 + 1] = src[index1 * ld_src + k];
    }
  }
  if (N % 2 != 0) {
    index_t index = ind[N - 1];
    for (int k = 0; k < Kv; ++k) {
      dst1[(N >> 1) * ld_dst1 * 2 + k * 2 + 0] = src[index * ld_src + k];
      dst1[(N >> 1) * ld_dst1 * 2 + k * 2 + 1] = 0;
    }
  }
#endif
}

template <typename scalar_t>
inline void fill_stub(scalar_t* __restrict__ out, float val, int64_t size) {
  using Vec = at::vec::Vectorized<scalar_t>;
  constexpr int kVecSize = Vec::size();
  const Vec data_vec = Vec(static_cast<scalar_t>(val));
  int64_t d = 0;
#pragma GCC unroll 4
  for (; d <= size - kVecSize; d += kVecSize) {
    data_vec.store(out + d);
  }
```
**EN:** This section uses `fill_stub`, `Vec`, `store` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fill_stub`、`Vec`、`store`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 139-162: Runtime integration and dispatch
```cpp
  if (size - d > 0) {
    data_vec.store(out + d, size - d);
  }
}

template <typename scalar_t>
inline void copy_stub(scalar_t* __restrict__ out, const float* __restrict__ acc, float s, int64_t size) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;
  constexpr int kVecSize = bVec::size();
  const fVec s_fvec = fVec(s);
  int64_t d = 0;
#pragma GCC unroll 4
  for (; d <= size - kVecSize; d += kVecSize) {
    fVec a_fvec0 = fVec::loadu(acc + d) * s_fvec;
    fVec a_fvec1 = fVec::loadu(acc + d + fVec::size()) * s_fvec;
    bVec out_bvec = convert_from_float_ext<scalar_t>(a_fvec0, a_fvec1);
    out_bvec.store(out + d);
  }
  for (; d < size; ++d) {
    out[d] = static_cast<scalar_t>(acc[d] * s);
  }
}
```
**EN:** This section uses `copy_stub`, `store`, `fVec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`copy_stub`、`store`、`fVec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 163-193: Runtime integration and dispatch
```cpp
template <typename scalar_t>
inline void copy_stub(scalar_t* __restrict__ out, const scalar_t* __restrict__ src, int64_t size) {
  using bVec = at::vec::Vectorized<scalar_t>;
  constexpr int kVecSize = bVec::size();
  int64_t d = 0;
#pragma GCC unroll 4
  for (; d <= size - kVecSize; d += kVecSize) {
    bVec out_bvec = bVec::loadu(src + d);
    out_bvec.store(out + d);
  }
  for (; d < size; ++d) {
    out[d] = src[d];
  }
}

template <typename scalar_t, int BLOCK_N>
inline void copy_stub(scalar_t* __restrict__ out, const float* __restrict__ input) {
  static_assert(BLOCK_N % 32 == 0);
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;

  constexpr int COLS = BLOCK_N / 16;
  auto store = [&](auto i) {
    constexpr int col = i % COLS;
    // for COLS = 2, 4 use 512bit store
    if constexpr (col % 2 == 0) {
      fVec a_fvec0 = fVec::loadu(input + col * 16);
      fVec a_fvec1 = fVec::loadu(input + col * 16 + 16);
      bVec out_bvec = convert_from_float_ext<scalar_t>(a_fvec0, a_fvec1);
      out_bvec.store(out + col * 16);
    }
```
**EN:** This section uses `copy_stub`, `loadu`, `store` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`copy_stub`、`loadu`、`store`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 194-223: Types and data layout
```cpp
  };
  Unroll<COLS>{}(store);
}

// GEMM handles query @ key (indexed) x scale
//   A : [M, K]
//   B : [N, K] indexed
//   C : [M, N]
//
template <typename scalar_t, typename index_t, int BLOCK_M, int BLOCK_N>
struct tinygemm_kernel_nt {
  static inline void apply(
      const scalar_t* __restrict__ A,
      const scalar_t* __restrict__ B,
      float* __restrict__ C,
      const index_t* __restrict__ indices,
      float scale,
      int64_t lda,
      int64_t ldb,
      int64_t ldc,
      int64_t K,
      int64_t max_tokens) {
    for (int64_t m = 0; m < BLOCK_M; ++m) {
      for (int64_t n = 0; n < BLOCK_N; ++n) {
        float sum = 0.f;
        int64_t b_idx = indices[n];
        TORCH_CHECK(b_idx < max_tokens, "token index out of scope!");
        for (int64_t k = 0; k < K; ++k) {
          sum += scale * static_cast<float>(A[m * lda + k]) * static_cast<float>(B[b_idx * ldb + k]);
        }
```
**EN:** This section defines `tinygemm_kernel_nt`, `apply`, `TORCH_CHECK`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`tinygemm_kernel_nt`、`apply`、`TORCH_CHECK`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 224-251: Types and data layout
```cpp
        C[m * ldc + n] = sum;
      }
    }
  }
};

#if defined(CPU_CAPABILITY_AVX512)
template <typename index_t, int BLOCK_M, int BLOCK_N>
struct tinygemm_kernel_nt<at::BFloat16, index_t, BLOCK_M, BLOCK_N> {
  static inline void apply(
      const at::BFloat16* __restrict__ A,
      const at::BFloat16* __restrict__ B,
      float* __restrict__ C,
      const index_t* __restrict__ indices,
      float scale,
      int64_t lda,
      int64_t ldb,
      int64_t ldc,
      int64_t K,
      int64_t max_tokens) {
    constexpr int ROWS = BLOCK_M;
    constexpr int COLS = BLOCK_N;

    __m512bh va;
    __m512bh vb[COLS];
    __m512 vc[ROWS * COLS];
    __m512 vscale = _mm512_set1_ps(scale);
```
**EN:** This section defines `tinygemm_kernel_nt`, `apply`, `_mm512_set1_ps`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`tinygemm_kernel_nt`、`apply`、`_mm512_set1_ps`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 252-279: Runtime integration and dispatch
```cpp
    auto loadc = [&](auto i) { vc[i] = _mm512_setzero_ps(); };
    Unroll<ROWS * COLS>{}(loadc);

    // for main loop
    auto compute = [&](auto i, int64_t k) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;

      if constexpr (col == 0) {
        va = (__m512bh)(_mm512_loadu_si512(A + row * lda + k));
      }
      if constexpr (row == 0) {
        if constexpr (col + 1 < COLS) {
          int64_t b_idx_prefetch = indices[col + 1];
          _mm_prefetch(B + b_idx_prefetch * ldb + k, _MM_HINT_T0);
        }
        int64_t b_idx = indices[col];
        TORCH_CHECK(b_idx < max_tokens, "token index out of scope!");
        vb[col] = (__m512bh)(_mm512_loadu_si512(B + b_idx * ldb + k));
      }
      vc[i] = _mm512_dpbf16_ps(vc[i], va, vb[col]);
    };

    // for remainder
    auto compute2 = [&](auto i, int64_t k, __mmask32 mask) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;
```
**EN:** This section uses `_mm512_setzero_ps`, `_mm512_loadu_si512`, `_mm_prefetch` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`_mm512_setzero_ps`、`_mm512_loadu_si512`、`_mm_prefetch`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 280-307: Runtime integration and dispatch
```cpp
      if constexpr (col == 0) {
        va = (__m512bh)(_mm512_maskz_loadu_epi16(mask, A + row * lda + k));
      }
      if constexpr (row == 0) {
        int64_t b_idx = indices[col];
        TORCH_CHECK(b_idx < max_tokens, "token index out of scope!");
        vb[col] = (__m512bh)(_mm512_maskz_loadu_epi16(mask, B + b_idx * ldb + k));
      }
      vc[i] = _mm512_dpbf16_ps(vc[i], va, vb[col]);
    };

    int64_t k = 0;
    for (; k <= K - 32; k += 32) {
      Unroll<ROWS * COLS>{}(compute, k);
    }
    int64_t count = K - k;
    if (count > 0) {
      __mmask32 mask = (1ULL << count) - 1;
      Unroll<ROWS * COLS>{}(compute2, k, mask);
    }

    auto storec = [&](auto i) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;
      C[row * ldc + col] = _mm512_reduce_add_ps(_mm512_mul_ps(vc[i], vscale));
    };
    Unroll<ROWS * COLS>{}(storec);
  }
```
**EN:** This section uses `_mm512_maskz_loadu_epi16`, `TORCH_CHECK`, `_mm512_dpbf16_ps` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`_mm512_maskz_loadu_epi16`、`TORCH_CHECK`、`_mm512_dpbf16_ps`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 308-335: Types and data layout
```cpp
};
#endif

#if defined(CPU_CAPABILITY_AVX512)
template <typename index_t, int BLOCK_M, int BLOCK_N>
struct tinygemm_kernel_nt<at::Half, index_t, BLOCK_M, BLOCK_N> {
  static inline void apply(
      const at::Half* __restrict__ A,
      const at::Half* __restrict__ B,
      float* __restrict__ C,
      const index_t* __restrict__ indices,
      float scale,
      int64_t lda,
      int64_t ldb,
      int64_t ldc,
      int64_t K,
      int64_t max_tokens) {
    constexpr int ROWS = BLOCK_M;
    constexpr int COLS = BLOCK_N;

    __m512 va0, va1;
    __m512 vb0[COLS], vb1[COLS];
    __m512 vc[ROWS * COLS];
    __m512 vscale = _mm512_set1_ps(scale);

    auto loadc = [&](auto i) { vc[i] = _mm512_setzero_ps(); };
    Unroll<ROWS * COLS>{}(loadc);
```
**EN:** This section defines `tinygemm_kernel_nt`, `apply`, `_mm512_set1_ps`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`tinygemm_kernel_nt`、`apply`、`_mm512_set1_ps`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 336-365: Runtime integration and dispatch
```cpp
    auto compute = [&](auto i, int64_t k) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;

      if constexpr (col == 0) {
        __m512i a16 = _mm512_loadu_si512((__m512i const*)(A + row * lda + k));
        va0 = CVT_FP16_TO_FP32(_mm512_extracti32x8_epi32(a16, 0));
        va1 = CVT_FP16_TO_FP32(_mm512_extracti32x8_epi32(a16, 1));
      }

      if constexpr (row == 0) {
        int64_t b_idx = indices[col];
        TORCH_CHECK(b_idx < max_tokens, "token index out of scope!");
        __m512i b16 = _mm512_loadu_si512((__m512i const*)(B + b_idx * ldb + k));
        vb0[col] = CVT_FP16_TO_FP32(_mm512_extracti32x8_epi32(b16, 0));
        vb1[col] = CVT_FP16_TO_FP32(_mm512_extracti32x8_epi32(b16, 1));
      }

      vc[i] = _mm512_fmadd_ps(va0, vb0[col], _mm512_fmadd_ps(va1, vb1[col], vc[i]));
    };

    auto compute2 = [&](auto i, int64_t k, __mmask32 mask) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;

      if constexpr (col == 0) {
        __m512i a16 = _mm512_maskz_loadu_epi16(mask, (const void*)(A + row * lda + k));
        va0 = CVT_FP16_TO_FP32(_mm512_extracti32x8_epi32(a16, 0));
        va1 = CVT_FP16_TO_FP32(_mm512_extracti32x8_epi32(a16, 1));
      }
```
**EN:** This section uses `_mm512_loadu_si512`, `CVT_FP16_TO_FP32`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`_mm512_loadu_si512`、`CVT_FP16_TO_FP32`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 366-394: Runtime integration and dispatch
```cpp

      if constexpr (row == 0) {
        int64_t b_idx = indices[col];
        TORCH_CHECK(b_idx < max_tokens, "token index out of scope!");
        __m512i b16 = _mm512_maskz_loadu_epi16(mask, (const void*)(B + b_idx * ldb + k));
        vb0[col] = CVT_FP16_TO_FP32(_mm512_extracti32x8_epi32(b16, 0));
        vb1[col] = CVT_FP16_TO_FP32(_mm512_extracti32x8_epi32(b16, 1));
      }

      vc[i] = _mm512_fmadd_ps(va0, vb0[col], _mm512_fmadd_ps(va1, vb1[col], vc[i]));
    };

    int64_t k = 0;
    for (; k <= K - 32; k += 32) {
      Unroll<ROWS * COLS>{}(compute, k);
    }
    int64_t count = K - k;
    if (count > 0) {
      __mmask32 mask = (1ULL << count) - 1;
      Unroll<ROWS * COLS>{}(compute2, k, mask);
    }

    auto storec = [&](auto i) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;
      C[row * ldc + col] = _mm512_reduce_add_ps(_mm512_mul_ps(vc[i], vscale));
    };
    Unroll<ROWS * COLS>{}(storec);
  }
```
**EN:** This section uses `TORCH_CHECK`, `_mm512_maskz_loadu_epi16`, `CVT_FP16_TO_FP32` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`、`_mm512_maskz_loadu_epi16`、`CVT_FP16_TO_FP32`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 395-425: Runtime integration and dispatch
```cpp
};
#endif

#define LAUNCH_TINYGEMM_KERNEL_NT(MB_SIZE, NB_SIZE)               \
  tinygemm_kernel_nt<scalar_t, index_t, MB_SIZE, NB_SIZE>::apply( \
      A + mb_start * lda, B, C + mb_start * ldc + nb_start, indices + nb_start, scale, lda, ldb, ldc, K, max_tokens);

// this is used when N isn't multiple of 16,
// N corresponds to `head_size_v` which should be 16x
template <typename scalar_t, typename index_t>
inline void tinygemm_kernel_nn_scalar(
    const float* __restrict__ A,
    const scalar_t* __restrict__ B,
    float* __restrict__ C,
    const index_t* __restrict__ indices,
    const float* __restrict__ scale,
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t lda,
    int64_t ldb,
    int64_t ldc,
    int64_t max_tokens) {
  for (int64_t m = 0; m < M; ++m) {
    for (int64_t n = 0; n < N; ++n) {
      C[m * ldc + n] *= scale[m];
      for (int64_t k = 0; k < K; ++k) {
        int64_t b_idx = indices[k];
        TORCH_CHECK(b_idx < max_tokens, "token index out of scope!");
        C[m * ldc + n] += A[m * lda + k] * static_cast<float>(B[b_idx * ldb + n]);
      }
```
**EN:** This section uses `tinygemm_kernel_nn_scalar`, `LAUNCH_TINYGEMM_KERNEL_NT`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`tinygemm_kernel_nn_scalar`、`LAUNCH_TINYGEMM_KERNEL_NT`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 426-451: Types and data layout
```cpp
    }
  }
}

// GEMM handles v' * scale + attn @ value (indexed)
//   A : [M, K]
//   B : [K, N] indexed
//   C ：[M, N]
//
template <typename scalar_t, typename index_t, int BLOCK_M, int BLOCK_N>
struct tinygemm_kernel_nn {
  static inline void apply(
      const float* __restrict__ A,
      const scalar_t* __restrict__ B,
      float* __restrict__ C,
      const index_t* __restrict__ indices,
      const float* __restrict__ scale,
      int64_t lda,
      int64_t ldb,
      int64_t ldc,
      int64_t K,
      int64_t max_tokens) {
    tinygemm_kernel_nn_scalar(A, B, C, indices, scale, BLOCK_M, BLOCK_N, K, lda, ldb, ldc, max_tokens);
  }
};
```
**EN:** This section defines `tinygemm_kernel_nn`, `apply`, `tinygemm_kernel_nn_scalar`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`tinygemm_kernel_nn`、`apply`、`tinygemm_kernel_nn_scalar`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 452-481: Types and data layout
```cpp
#if defined(CPU_CAPABILITY_AVX512)
template <typename index_t, int BLOCK_M, int BLOCK_N>
struct tinygemm_kernel_nn<at::BFloat16, index_t, BLOCK_M, BLOCK_N> {
  static inline void apply(
      const float* __restrict__ A,
      const at::BFloat16* __restrict__ B,
      float* __restrict__ C,
      const index_t* __restrict__ indices,
      const float* __restrict__ scale,
      int64_t lda,
      int64_t ldb,
      int64_t ldc,
      int64_t K,
      int64_t max_tokens) {
    constexpr int ROWS = BLOCK_M;
    constexpr int COLS = BLOCK_N / 16;

    __m512 va;
    __m512 vb[COLS];
    __m512 vc[ROWS * COLS];
    __m512 vscale;

    auto loadc = [&](auto i) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;
#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Warray-bounds"
      if constexpr (col == 0) {
        vscale = _mm512_set1_ps(scale[row]);
      }
```
**EN:** This section defines `tinygemm_kernel_nn`, `apply`, `_mm512_set1_ps`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`tinygemm_kernel_nn`、`apply`、`_mm512_set1_ps`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 482-510: Runtime integration and dispatch
```cpp
#pragma GCC diagnostic pop
      vc[i] = _mm512_loadu_ps(C + row * ldc + col * 16);
      vc[i] = _mm512_mul_ps(vc[i], vscale);
    };
    Unroll<ROWS * COLS>{}(loadc);

    auto compute = [&](auto i, int64_t k) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;

      if constexpr (col == 0) {
        va = _mm512_set1_ps(A[row * lda + k]);
      }
      if constexpr (row == 0) {
        if (k + 1 < K) {
          int64_t b_idx_prefetch = indices[k + 1];
          _mm_prefetch(B + b_idx_prefetch * ldb + col * 16, _MM_HINT_T0);
        }
        int64_t b_idx = indices[k];
        TORCH_CHECK(b_idx < max_tokens, "token index out of scope!");

        // for COLS = 2, 4, 6, 8 use 512 bit load
        // for COLS = 1, 3, 5, 7 use 256 bit load
        if constexpr (COLS % 2 == 0) {
          if constexpr (col % 2 == 0) {
            __m512i b16 = _mm512_loadu_si512(reinterpret_cast<const __m512i*>(B + b_idx * ldb + col * 16));
            vb[col + 0] = CVT_BF16_TO_FP32(_mm512_extracti32x8_epi32(b16, 0));
            vb[col + 1] = CVT_BF16_TO_FP32(_mm512_extracti32x8_epi32(b16, 1));
          }
```
**EN:** This section uses `_mm512_loadu_ps`, `_mm512_mul_ps`, `_mm512_set1_ps` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`_mm512_loadu_ps`、`_mm512_mul_ps`、`_mm512_set1_ps`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 511-532: Templates, aliases, and constants
```cpp
        } else {
          __m256i b16 = _mm256_loadu_si256(reinterpret_cast<const __m256i*>(B + b_idx * ldb + col * 16));
          vb[col] = CVT_BF16_TO_FP32(b16);
        }
      }
      vc[i] = _mm512_fmadd_ps(va, vb[col], vc[i]);
    };

    for (int64_t k = 0; k < K; ++k) {
      Unroll<ROWS * COLS>{}(compute, k);
    }

    auto storec = [&](auto i) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;
      _mm512_storeu_ps(C + row * ldc + col * 16, vc[i]);
    };
    Unroll<ROWS * COLS>{}(storec);
  }
};
#endif
```
**EN:** This section defines `_mm256_loadu_si256`, `CVT_BF16_TO_FP32`, `_mm512_fmadd_ps`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`_mm256_loadu_si256`、`CVT_BF16_TO_FP32`、`_mm512_fmadd_ps`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 533-562: Types and data layout
```cpp
#if defined(CPU_CAPABILITY_AVX512)
template <typename index_t, int BLOCK_M, int BLOCK_N>
struct tinygemm_kernel_nn<at::Half, index_t, BLOCK_M, BLOCK_N> {
  static inline void apply(
      const float* __restrict__ A,
      const at::Half* __restrict__ B,
      float* __restrict__ C,
      const index_t* __restrict__ indices,
      const float* __restrict__ scale,
      int64_t lda,
      int64_t ldb,
      int64_t ldc,
      int64_t K,
      int64_t max_tokens) {
    constexpr int ROWS = BLOCK_M;
    constexpr int COLS = BLOCK_N / 16;

    __m512 va;
    __m512 vb[COLS];
    __m512 vc[ROWS * COLS];
    __m512 vscale;

    auto loadc = [&](auto i) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;
#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Warray-bounds"
      if constexpr (col == 0) {
        vscale = _mm512_set1_ps(scale[row]);
      }
```
**EN:** This section defines `tinygemm_kernel_nn`, `apply`, `_mm512_set1_ps`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`tinygemm_kernel_nn`、`apply`、`_mm512_set1_ps`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 563-591: Runtime integration and dispatch
```cpp
#pragma GCC diagnostic pop
      vc[i] = _mm512_loadu_ps(C + row * ldc + col * 16);
      vc[i] = _mm512_mul_ps(vc[i], vscale);
    };
    Unroll<ROWS * COLS>{}(loadc);

    auto compute = [&](auto i, int64_t k) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;

      if constexpr (col == 0) {
        va = _mm512_set1_ps(A[row * lda + k]);
      }
      if constexpr (row == 0) {
        if (k + 1 < K) {
          int64_t b_idx_prefetch = indices[k + 1];
          _mm_prefetch(B + b_idx_prefetch * ldb + col * 16, _MM_HINT_T0);
        }
        int64_t b_idx = indices[k];
        TORCH_CHECK(b_idx < max_tokens, "token index out of scope!");

        // for COLS = 2, 4, 6, 8 use 512 bit load
        // for COLS = 1, 3, 5, 7 use 256 bit load
        if constexpr (COLS % 2 == 0) {
          if constexpr (col % 2 == 0) {
            __m512i b16 = _mm512_loadu_si512(reinterpret_cast<const __m512i*>(B + b_idx * ldb + col * 16));
            vb[col + 0] = CVT_FP16_TO_FP32(_mm512_extracti32x8_epi32(b16, 0));
            vb[col + 1] = CVT_FP16_TO_FP32(_mm512_extracti32x8_epi32(b16, 1));
          }
```
**EN:** This section uses `_mm512_loadu_ps`, `_mm512_mul_ps`, `_mm512_set1_ps` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`_mm512_loadu_ps`、`_mm512_mul_ps`、`_mm512_set1_ps`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 592-613: Templates, aliases, and constants
```cpp
        } else {
          __m256i b16 = _mm256_loadu_si256(reinterpret_cast<const __m256i*>(B + b_idx * ldb + col * 16));
          vb[col] = CVT_FP16_TO_FP32(b16);
        }
      }
      vc[i] = _mm512_fmadd_ps(va, vb[col], vc[i]);
    };

    for (int64_t k = 0; k < K; ++k) {
      Unroll<ROWS * COLS>{}(compute, k);
    }

    auto storec = [&](auto i) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;
      _mm512_storeu_ps(C + row * ldc + col * 16, vc[i]);
    };
    Unroll<ROWS * COLS>{}(storec);
  }
};
#endif
```
**EN:** This section defines `_mm256_loadu_si256`, `CVT_FP16_TO_FP32`, `_mm512_fmadd_ps`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`_mm256_loadu_si256`、`CVT_FP16_TO_FP32`、`_mm512_fmadd_ps`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 614-646: Templates, aliases, and constants
```cpp
#define LAUNCH_TINYGEMM_KERNEL_NN(MB_SIZE, NB_SIZE)               \
  tinygemm_kernel_nn<scalar_t, index_t, MB_SIZE, NB_SIZE>::apply( \
      A + mb_start * lda,                                         \
      B + nb_start,                                               \
      C + mb_start * ldc + nb_start,                              \
      indices,                                                    \
      scale + mb_start,                                           \
      lda,                                                        \
      ldb,                                                        \
      ldc,                                                        \
      K,                                                          \
      max_tokens);

template <typename scalar_t, typename index_t>
void index_gemm_kernel_nt(
    const scalar_t* __restrict__ A,
    const scalar_t* __restrict__ B,
    float* __restrict__ C,
    const index_t* __restrict__ indices,
    float scale,
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t lda,
    int64_t ldb,
    int64_t ldc,
    int64_t max_tokens) {
  // pattern: 1-8-8
  if (M == 1) {
    constexpr int64_t BLOCK_N = 8;
    const int64_t NB = div_up(N, BLOCK_N);
    int64_t mb_start = 0, lda = 1, ldc = 1;
```
**EN:** This section defines `index_gemm_kernel_nt`, `LAUNCH_TINYGEMM_KERNEL_NN`, `div_up`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`index_gemm_kernel_nt`、`LAUNCH_TINYGEMM_KERNEL_NN`、`div_up`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 647-678: Runtime integration and dispatch
```cpp
    for (int64_t nb = 0; nb < NB; ++nb) {
      int64_t nb_start = nb * BLOCK_N;
      int64_t nb_size = std::min(BLOCK_N, N - nb_start);

      switch (nb_size) {
        case 1:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 1);
          break;
        case 2:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 2);
          break;
        case 3:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 3);
          break;
        case 4:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 4);
          break;
        case 5:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 5);
          break;
        case 6:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 6);
          break;
        case 7:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 7);
          break;
        case 8:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 8);
          break;
        default:
          TORCH_CHECK(false, "Unexpected block size, 1x", "nb_size");
      }
```
**EN:** This section uses `min`, `LAUNCH_TINYGEMM_KERNEL_NT`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`min`、`LAUNCH_TINYGEMM_KERNEL_NT`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 679-706: Runtime integration and dispatch
```cpp
    }
    return;
  }

  // default pattern: 1-6-24
  // FP16 pattern: 2-8-16
  constexpr int64_t BLOCK_M = 4;
  constexpr int64_t BLOCK_N = std::is_same_v<scalar_t, at::Half> ? 4 : 6;
  const int64_t MB = div_up(M, BLOCK_M);
  const int64_t NB = div_up(N, BLOCK_N);

  for (int64_t mb = 0; mb < MB; ++mb) {
    int64_t mb_start = mb * BLOCK_M;
    int64_t mb_size = std::min(BLOCK_M, M - mb_start);
    for (int64_t nb = 0; nb < NB; ++nb) {
      int64_t nb_start = nb * BLOCK_N;
      int64_t nb_size = std::min(BLOCK_N, N - nb_start);

      switch (mb_size << 4 | nb_size) {
        // mb_size = 1
        case 0x11:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 1);
          break;
        case 0x12:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 2);
          break;
        case 0x13:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 3);
```
**EN:** This section uses `div_up`, `min`, `LAUNCH_TINYGEMM_KERNEL_NT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`div_up`、`min`、`LAUNCH_TINYGEMM_KERNEL_NT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 707-734: Control flow and branching
```cpp
          break;
        case 0x14:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 4);
          break;
        case 0x15:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 5);
          break;
        case 0x16:
          LAUNCH_TINYGEMM_KERNEL_NT(1, 6);
          break;
        // mb_size = 2
        case 0x21:
          LAUNCH_TINYGEMM_KERNEL_NT(2, 1);
          break;
        case 0x22:
          LAUNCH_TINYGEMM_KERNEL_NT(2, 2);
          break;
        case 0x23:
          LAUNCH_TINYGEMM_KERNEL_NT(2, 3);
          break;
        case 0x24:
          LAUNCH_TINYGEMM_KERNEL_NT(2, 4);
          break;
        case 0x25:
          LAUNCH_TINYGEMM_KERNEL_NT(2, 5);
          break;
        case 0x26:
          LAUNCH_TINYGEMM_KERNEL_NT(2, 6);
```
**EN:** This section drives `LAUNCH_TINYGEMM_KERNEL_NT` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`LAUNCH_TINYGEMM_KERNEL_NT`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 735-762: Control flow and branching
```cpp
          break;
        // mb_size = 3
        case 0x31:
          LAUNCH_TINYGEMM_KERNEL_NT(3, 1);
          break;
        case 0x32:
          LAUNCH_TINYGEMM_KERNEL_NT(3, 2);
          break;
        case 0x33:
          LAUNCH_TINYGEMM_KERNEL_NT(3, 3);
          break;
        case 0x34:
          LAUNCH_TINYGEMM_KERNEL_NT(3, 4);
          break;
        case 0x35:
          LAUNCH_TINYGEMM_KERNEL_NT(3, 5);
          break;
        case 0x36:
          LAUNCH_TINYGEMM_KERNEL_NT(3, 6);
          break;
        // mb_size = 4
        case 0x41:
          LAUNCH_TINYGEMM_KERNEL_NT(4, 1);
          break;
        case 0x42:
          LAUNCH_TINYGEMM_KERNEL_NT(4, 2);
          break;
        case 0x43:
```
**EN:** This section drives `LAUNCH_TINYGEMM_KERNEL_NT` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`LAUNCH_TINYGEMM_KERNEL_NT`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 763-790: Runtime integration and dispatch
```cpp
          LAUNCH_TINYGEMM_KERNEL_NT(4, 3);
          break;
        case 0x44:
          LAUNCH_TINYGEMM_KERNEL_NT(4, 4);
          break;
        case 0x45:
          LAUNCH_TINYGEMM_KERNEL_NT(4, 5);
          break;
        case 0x46:
          LAUNCH_TINYGEMM_KERNEL_NT(4, 6);
          break;
        default:
          TORCH_CHECK(false, "Unexpected block size, ", mb_size, "x", "nb_size");
      }
    }
  }
}

template <typename scalar_t, typename index_t>
void index_gemm_kernel_nn(
    const float* __restrict__ A,
    const scalar_t* __restrict__ B,
    float* __restrict__ C,
    const index_t* __restrict__ indices,
    float* __restrict__ scale,
    int64_t M,
    int64_t N,
    int64_t K,
```
**EN:** This section uses `LAUNCH_TINYGEMM_KERNEL_NT`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`LAUNCH_TINYGEMM_KERNEL_NT`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 791-818: Templates, aliases, and constants
```cpp
    int64_t lda,
    int64_t ldb,
    int64_t ldc,
    int64_t max_tokens) {
  constexpr int kVecSize = 16;
  if ((N & (kVecSize - 1)) != 0) {
    tinygemm_kernel_nn_scalar(A, B, C, indices, scale, M, N, K, lda, ldb, ldc, max_tokens);
    return;
  }

  // pattern: 1-8-8
  if (M == 1) {
    constexpr int64_t BLOCK_N = 8 * kVecSize;
    const int64_t NB = div_up(N, BLOCK_N);
    int64_t mb_start = 0, lda = 1, ldc = 1;

    for (int64_t nb = 0; nb < NB; ++nb) {
      int64_t nb_start = nb * BLOCK_N;
      int64_t nb_size = std::min(BLOCK_N, N - nb_start);

      switch (nb_size >> 4) {
        case 1:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 16);
          break;
        case 2:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 32);
          break;
        case 3:
```
**EN:** This section defines `tinygemm_kernel_nn_scalar`, `div_up`, `min`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`tinygemm_kernel_nn_scalar`、`div_up`、`min`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 819-847: Runtime integration and dispatch
```cpp
          LAUNCH_TINYGEMM_KERNEL_NN(1, 48);
          break;
        case 4:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 64);
          break;
        case 5:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 80);
          break;
        case 6:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 96);
          break;
        case 7:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 112);
          break;
        case 8:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 128);
          break;
        default:
          TORCH_CHECK(false, "Unexpected block size, 1x", "nb_size");
      }
    }
    return;
  }

  constexpr int64_t BLOCK_M = 4;
  constexpr int64_t BLOCK_N = 6 * kVecSize;
  const int64_t MB = div_up(M, BLOCK_M);
  const int64_t NB = div_up(N, BLOCK_N);
```
**EN:** This section uses `LAUNCH_TINYGEMM_KERNEL_NN`, `TORCH_CHECK`, `div_up` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`LAUNCH_TINYGEMM_KERNEL_NN`、`TORCH_CHECK`、`div_up`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 848-875: Control flow and branching
```cpp
  for (int64_t mb = 0; mb < MB; ++mb) {
    int64_t mb_start = mb * BLOCK_M;
    int64_t mb_size = std::min(BLOCK_M, M - mb_start);
    for (int64_t nb = 0; nb < NB; ++nb) {
      int64_t nb_start = nb * BLOCK_N;
      int64_t nb_size = std::min(BLOCK_N, N - nb_start);

      switch (mb_size << 4 | nb_size >> 4) {
        // mb_size = 1
        case 0x11:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 16);
          break;
        case 0x12:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 32);
          break;
        case 0x13:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 48);
          break;
        case 0x14:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 64);
          break;
        case 0x15:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 80);
          break;
        case 0x16:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 96);
          break;
        // mb_size = 2
```
**EN:** This section drives `min`, `LAUNCH_TINYGEMM_KERNEL_NN` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`min`、`LAUNCH_TINYGEMM_KERNEL_NN`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 876-903: Control flow and branching
```cpp
        case 0x21:
          LAUNCH_TINYGEMM_KERNEL_NN(2, 16);
          break;
        case 0x22:
          LAUNCH_TINYGEMM_KERNEL_NN(2, 32);
          break;
        case 0x23:
          LAUNCH_TINYGEMM_KERNEL_NN(2, 48);
          break;
        case 0x24:
          LAUNCH_TINYGEMM_KERNEL_NN(2, 64);
          break;
        case 0x25:
          LAUNCH_TINYGEMM_KERNEL_NN(2, 80);
          break;
        case 0x26:
          LAUNCH_TINYGEMM_KERNEL_NN(2, 96);
          break;
        // mb_size = 3
        case 0x31:
          LAUNCH_TINYGEMM_KERNEL_NN(3, 16);
          break;
        case 0x32:
          LAUNCH_TINYGEMM_KERNEL_NN(3, 32);
          break;
        case 0x33:
          LAUNCH_TINYGEMM_KERNEL_NN(3, 48);
          break;
```
**EN:** This section drives `LAUNCH_TINYGEMM_KERNEL_NN` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`LAUNCH_TINYGEMM_KERNEL_NN`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 904-934: Runtime integration and dispatch
```cpp
        case 0x34:
          LAUNCH_TINYGEMM_KERNEL_NN(3, 64);
          break;
        case 0x35:
          LAUNCH_TINYGEMM_KERNEL_NN(3, 80);
          break;
        case 0x36:
          LAUNCH_TINYGEMM_KERNEL_NN(3, 96);
          break;
        // mb_size = 4
        case 0x41:
          LAUNCH_TINYGEMM_KERNEL_NN(4, 16);
          break;
        case 0x42:
          LAUNCH_TINYGEMM_KERNEL_NN(4, 32);
          break;
        case 0x43:
          LAUNCH_TINYGEMM_KERNEL_NN(4, 48);
          break;
        case 0x44:
          LAUNCH_TINYGEMM_KERNEL_NN(4, 64);
          break;
        case 0x45:
          LAUNCH_TINYGEMM_KERNEL_NN(4, 80);
          break;
        case 0x46:
          LAUNCH_TINYGEMM_KERNEL_NN(4, 96);
          break;
        default:
          TORCH_CHECK(false, "Unexpected block size, ", mb_size, "x", "nb_size");
      }
```
**EN:** This section uses `LAUNCH_TINYGEMM_KERNEL_NN`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`LAUNCH_TINYGEMM_KERNEL_NN`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 935-962: Runtime integration and dispatch
```cpp
    }
  }
}

template <typename scalar_t>
void decode_set_kv_buffer(
    scalar_t* __restrict__ k_buffer,
    scalar_t* __restrict__ v_buffer,
    const scalar_t* __restrict__ key,
    const scalar_t* __restrict__ value,
    const int64_t* __restrict__ loc,
    int64_t batches,
    int64_t num_heads_kv,
    int64_t head_size,
    int64_t head_size_v,
    int64_t k_strideN,
    int64_t k_strideH,
    int64_t v_strideN,
    int64_t v_strideH,
    int64_t nk_strideN,
    int64_t nk_strideH,
    int64_t nv_strideN,
    int64_t nv_strideH,
    bool is_mla) {
  at::parallel_for(0, batches * num_heads_kv, 0, [&](int64_t begin, int64_t end) {
    int64_t bs{0}, head_kv_id{0};
    data_index_init(begin, bs, batches, head_kv_id, num_heads_kv);
```
**EN:** This section uses `decode_set_kv_buffer`, `at::parallel_for`, `data_index_init` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`decode_set_kv_buffer`、`at::parallel_for`、`data_index_init`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 963-991: Runtime integration and dispatch
```cpp
    for (int64_t i = begin; i < end; i++) {
      int64_t loc_val = loc[bs];
      scalar_t* k_buffer_ptr = k_buffer + loc_val * k_strideN + head_kv_id * k_strideH;
      const scalar_t* new_key_ptr = key + bs * nk_strideN + head_kv_id * nk_strideH;
      copy_stub<scalar_t>(k_buffer_ptr, new_key_ptr, head_size);
      if (!is_mla) {
        scalar_t* v_buffer_ptr = v_buffer + loc_val * v_strideN + head_kv_id * v_strideH;
        const scalar_t* new_value_ptr = value + bs * nv_strideN + head_kv_id * nv_strideH;
        copy_stub<scalar_t>(v_buffer_ptr, new_value_ptr, head_size_v);
      }

      // move to the next index
      data_index_step(bs, batches, head_kv_id, num_heads_kv);
    }
  });
}

template <typename scalar_t>
void decode_accumulate_kv_splits(
    scalar_t* __restrict__ output,
    float* __restrict__ attn_logits,
    int64_t batches,
    int64_t num_heads,
    int64_t head_size_v,
    int64_t num_kv_splits,
    int64_t l_stride1,
    int64_t l_stride2) {
  using Vec = at::vec::Vectorized<float>;
```
**EN:** This section uses `decode_accumulate_kv_splits`, `data_index_step`, `new_key_ptr` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`decode_accumulate_kv_splits`、`data_index_step`、`new_key_ptr`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 992-1020: Runtime integration and dispatch
```cpp
  // parallel on [batches, num_heads]
  at::parallel_for(0, batches * num_heads, 0, [&](int64_t begin, int64_t end) {
    // NB: here we use logits[b][h][0] as acc, since
    // for the first kv split (kv_id == 0):
    //   m_delta = std::exp(-inf) = 0
    //   e_logic = std::exp(0) = 1
    //   acc = acc * m_delta + tv * e_logic = tv
    for (int64_t i = begin; i < end; ++i) {
      float* __restrict__ acc = attn_logits + i * l_stride1;

      float s_prime = 0.f;
      float m_prime = -std::numeric_limits<scalar_t>::infinity();

      // update acc with from each kv_split
      for (int64_t kv_id = 0; kv_id < num_kv_splits; ++kv_id) {
        float* __restrict__ tv = acc + kv_id * l_stride2;
        const float tlogic = (acc + kv_id * l_stride2)[head_size_v];

        float m_i = std::max(tlogic, m_prime);
        float m_delta = std::exp(m_prime - m_i);
        float e_logic = std::exp(tlogic - m_i);
        if (kv_id != 0) {
          at::vec::map2<float>(
              [m_delta, e_logic](Vec x, Vec y) { return x * Vec(m_delta) + y * Vec(e_logic); },
              acc,
              acc,
              tv,
              head_size_v);
        }
```
**EN:** This section uses `at::parallel_for`, `infinity`, `max` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`infinity`、`max`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1021-1048: Templates, aliases, and constants
```cpp

        s_prime = s_prime * m_delta + e_logic;
        m_prime = m_i;
      }

      copy_stub<scalar_t>(output + i * head_size_v, acc, 1 / s_prime, head_size_v);
    }
  });
}

template <typename scalar_t, typename index_t, int64_t BLOCK_N>
void decode_attention_kernel_impl(
    scalar_t* __restrict__ output,
    float* __restrict__ attn_logits,
    const scalar_t* __restrict__ query,
    const scalar_t* __restrict__ k_buffer,
    const scalar_t* __restrict__ v_buffer,
    const index_t* __restrict__ req_to_token,
    const int64_t* __restrict__ req_pool_indices,
    const int64_t* __restrict__ seq_lens,
    int64_t batches,
    int64_t num_heads,
    int64_t head_size,
    int64_t head_size_v,
    int64_t num_kv_splits,
    int64_t q_strideM,
    int64_t q_strideH,
    int64_t k_strideN,
```
**EN:** This section defines the surrounding logic, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了相关逻辑等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 1049-1078: Runtime integration and dispatch
```cpp
    int64_t k_strideH,
    int64_t v_strideN,
    int64_t v_strideH,
    float sm_scale,
    float logit_cap,
    int64_t max_num_reqs,
    int64_t max_context_len,
    int64_t max_total_num_tokens) {
  using Vec = at::vec::Vectorized<float>;

  // strides
  const int64_t l_stride1 = num_kv_splits * (head_size_v + 1);
  const int64_t l_stride2 = head_size_v + 1;

  const bool has_logit_cap = logit_cap > 0;
  float rlogit_cap = has_logit_cap ? 1 / logit_cap : 0.f;

  // parallel on [batches, num_heads, num_kv_splits]
  at::parallel_for(0, batches * num_heads * num_kv_splits, 0, [&](int64_t begin, int64_t end) {
    int64_t bs{0}, head_id{0}, kv_id{0};
    data_index_init(begin, bs, batches, head_id, num_heads, kv_id, num_kv_splits);

    // s_prime and s_delta
    alignas(64) float s_i[BLOCK_N];
    float* __restrict__ s_delta = s_i;

    for (int64_t i = begin; i < end; ++i) {
      // get query
      const scalar_t* __restrict__ q_ptr = query + bs * q_strideM + head_id * q_strideH;
```
**EN:** This section uses `at::parallel_for`, `data_index_init`, `Vec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`data_index_init`、`Vec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1079-1106: Runtime integration and dispatch
```cpp
      // get key/value
      int64_t seq_len_kv = seq_lens[bs];
      int64_t req_pool_id = req_pool_indices[bs];
      TORCH_CHECK(seq_len_kv <= max_context_len, "seq_len_kv out of scope!");
      TORCH_CHECK(req_pool_id < max_num_reqs, "req_pool_id out of scope!");

      const int64_t SPLIT_SIZE = div_up(seq_len_kv, num_kv_splits);
      const int64_t kv_start = kv_id * SPLIT_SIZE;
      const int64_t kv_end = std::min(kv_start + SPLIT_SIZE, seq_len_kv);

      float m_prime = -std::numeric_limits<float>::infinity();
      float s_prime = 0.f;

      // get v_prime, and init to zero
      float* __restrict__ v_prime = attn_logits + i * (head_size_v + 1);
      fill_stub(v_prime, 0.f, head_size_v);

      // loop over K and V sequence with BLOCK_N
      for (int64_t n = kv_start; n < kv_end; n += BLOCK_N) {
        int64_t n_size = std::min(BLOCK_N, kv_end - n);

        // calculate s_i <- scale * Q @ K
        index_gemm_kernel_nt<scalar_t, index_t>(
            /* A   */ q_ptr,
            /* B   */ k_buffer + head_id * k_strideH,
            /* C   */ s_i,
            /* ind */ req_to_token + req_pool_id * max_context_len + n,
            /* scl */ sm_scale,
```
**EN:** This section uses `TORCH_CHECK`, `div_up`, `min` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`、`div_up`、`min`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1107-1133: Runtime integration and dispatch
```cpp
            /* M   */ 1,
            /* N   */ n_size,
            /* K   */ head_size,
            /* lda */ 1,
            /* ldb */ k_strideN,
            /* ldc */ 1,
            /* mtt */ max_total_num_tokens);

        // TODO: `tanh` from torch uses sleef u10, going to be slow
        if (has_logit_cap) {
          at::vec::map<float>(
              [logit_cap, rlogit_cap](Vec x) { return Vec(logit_cap) * (x * Vec(rlogit_cap)).tanh(); },
              s_i,
              s_i,
              n_size);
        }

        // m_i: max value per row
        float m_i = at::vec::reduce_all<float>([](Vec& x, Vec& y) { return at::vec::maximum(x, y); }, s_i, n_size);
        m_i = std::max(m_i, m_prime);

        // m_delta <- exp(m' - m_i)
        float m_delta = std::exp(m_prime - m_i);

        // s_delta <- exp(s_i - m_i)
        at::vec::map<float>([m_i](Vec x) { return (x - Vec(m_i)).exp_u20(); }, s_delta, s_i, n_size);
```
**EN:** This section uses `Vec`, `maximum`, `max` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`Vec`、`maximum`、`max`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1134-1162: Runtime integration and dispatch
```cpp
        // s' <- s' * m_delta + sum(s_delta)
        s_prime *= m_delta;
        s_prime += at::vec::reduce_all<float>([](Vec& x, Vec& y) { return x + y; }, s_delta, n_size);

        m_prime = m_i;

        // calculate V' <- s_delta @ V + V' * m_delta
        index_gemm_kernel_nn<scalar_t, index_t>(
            /* A   */ s_delta,
            /* B   */ v_buffer + head_id * v_strideH,
            /* C   */ v_prime,
            /* ind */ req_to_token + req_pool_id * max_context_len + n,
            /* scl */ &m_delta,
            /* M   */ 1,
            /* N   */ head_size_v,
            /* K   */ n_size,
            /* lda */ 1,
            /* ldb */ v_strideN,
            /* ldc */ 1,
            /* mtt */ max_total_num_tokens);
      }  // loop with KV blocks

      // only update v' when kv_split_size > 0
      if (kv_end > kv_start) {
        float s = 1 / s_prime;
        at::vec::map<float>([s](Vec out) { return out * Vec(s); }, v_prime, v_prime, head_size_v);

        v_prime[head_size_v] = m_prime + std::log(s_prime);
      }
```
**EN:** This section uses `Vec`, `log` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`Vec`、`log`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1163-1190: Templates, aliases, and constants
```cpp

      // move to the next index
      data_index_step(bs, batches, head_id, num_heads, kv_id, num_kv_splits);
    }
  });

  decode_accumulate_kv_splits(
      output, attn_logits, batches, num_heads, head_size_v, num_kv_splits, l_stride1, l_stride2);
}  // MHA

template <typename scalar_t, typename index_t, int64_t BLOCK_N>
void decode_attention_mla_kernel_impl(
    scalar_t* __restrict__ output,
    float* __restrict__ attn_logits,
    const scalar_t* __restrict__ query,
    const scalar_t* __restrict__ k_buffer,
    const scalar_t* __restrict__ v_buffer,
    const index_t* __restrict__ req_to_token,
    const int64_t* __restrict__ req_pool_indices,
    const int64_t* __restrict__ seq_lens,
    scalar_t* __restrict__ buffer,
    int64_t batches,
    int64_t num_heads,
    int64_t head_size,
    int64_t head_size_v,
    int64_t num_kv_splits,
    int64_t q_strideM,
    int64_t q_strideH,
```
**EN:** This section defines `data_index_step`, `decode_accumulate_kv_splits`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`data_index_step`、`decode_accumulate_kv_splits`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 1191-1220: Runtime integration and dispatch
```cpp
    int64_t k_strideN,
    int64_t k_strideH,
    int64_t v_strideN,
    int64_t v_strideH,
    float sm_scale,
    float logit_cap,
    int64_t max_num_reqs,
    int64_t max_context_len,
    int64_t max_total_num_tokens,
    int64_t buffer_size_per_thread) {
  using Vec = at::vec::Vectorized<float>;

  // block length for heads
  const int64_t BLOCK_H = batches == 1 ? 6 : (batches > 16 ? 22 : 11);

  // strides
  const int64_t l_stride0 = num_heads * num_kv_splits * (head_size_v + 1);
  const int64_t l_stride1 = num_kv_splits * (head_size_v + 1);
  const int64_t l_stride2 = head_size_v + 1;

  TORCH_CHECK(logit_cap == 0.f, "decode MLA: expect no logit_cap.");

  // partition the heads into blocks for parallel
  const int64_t num_blocks = div_up(num_heads, BLOCK_H);

  // parallel on [batches, num_blocks, num_kv_splits]
  at::parallel_for(0, batches * num_blocks * num_kv_splits, 0, [&](int64_t begin, int64_t end) {
    int64_t bs{0}, block_id{0}, kv_id{0};
    data_index_init(begin, bs, batches, block_id, num_blocks, kv_id, num_kv_splits);
```
**EN:** This section uses `at::parallel_for`, `TORCH_CHECK`, `div_up` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`TORCH_CHECK`、`div_up`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1221-1249: Runtime integration and dispatch
```cpp
    int tid = at::get_thread_num();
    scalar_t* __restrict__ Btmp0 = buffer + tid * buffer_size_per_thread;
    scalar_t* __restrict__ Btmp1 = Btmp0 + BLOCK_N * head_size;

    // init Btmp1 just once for each thread to prevent NaN
    // Btmp0 is not needed as it computes full K every single time
    fill_stub(Btmp1, 0.f, BLOCK_N * head_size_v);

    alignas(64) float s_i[BLOCK_H * BLOCK_N];
    float* __restrict__ s_delta = s_i;
    alignas(64) scalar_t s_delta2[BLOCK_H * BLOCK_N];

    alignas(64) float s_prime[BLOCK_H];
    alignas(64) float m_prime[BLOCK_H];
    alignas(64) float m_delta[BLOCK_H];

    for (int64_t i = begin; i < end; ++i) {
      const int64_t h_start = block_id * BLOCK_H;
      const int64_t h_end = std::min(block_id * BLOCK_H + BLOCK_H, num_heads);
      const int64_t h_size = h_end - h_start;

      // get query
      const scalar_t* __restrict__ q_ptr = query + bs * q_strideM + h_start * q_strideH;

      int64_t seq_len_kv = seq_lens[bs];
      int64_t req_pool_id = req_pool_indices[bs];
      TORCH_CHECK(seq_len_kv <= max_context_len, "seq_len_kv out of scope!");
      TORCH_CHECK(req_pool_id < max_num_reqs, "req_pool_id out of scope!");
```
**EN:** This section uses `get_thread_num`, `fill_stub`, `min` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`get_thread_num`、`fill_stub`、`min`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1250-1280: Control flow and branching
```cpp
      const int64_t SPLIT_SIZE = div_up(seq_len_kv, num_kv_splits);
      const int64_t kv_start = kv_id * SPLIT_SIZE;
      const int64_t kv_end = std::min(kv_start + SPLIT_SIZE, seq_len_kv);

      fill_stub(s_prime, 0.f, BLOCK_H);
      fill_stub(m_prime, -std::numeric_limits<float>::infinity(), BLOCK_H);

      // get v_prime, and init to zero
      float* __restrict__ v_prime = attn_logits + bs * l_stride0 + h_start * l_stride1 + kv_id * l_stride2;
      for (int64_t h = 0; h < h_size; ++h) {
        fill_stub(v_prime + h * l_stride1, 0.f, head_size_v);
      }

      // loop over K and V sequence with BLOCK_N
      for (int64_t n = kv_start; n < kv_end; n += BLOCK_N) {
        int64_t n_size = std::min(BLOCK_N, kv_end - n);
        const int64_t padded_n_size = div_up(int(n_size), TILE_K) * TILE_K;

        // get key and pack
        pack_vnni<scalar_t, index_t>(
            /*    dst0 */ Btmp0,
            /*    dst1 */ Btmp1,
            /*     src */ k_buffer + /* head_kv_id */ 0 * k_strideH,
            /*     ind */ req_to_token + req_pool_id * max_context_len + n,
            /*       N */ n_size,
            /*       K */ head_size,
            /*      Kv */ head_size_v,
            /*  ld_src */ k_strideN,
            /* ld_dst0 */ BLOCK_N,
            /* ld_dst1 */ head_size_v);
```
**EN:** This section drives `div_up`, `min`, `fill_stub` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`div_up`、`min`、`fill_stub`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1281-1307: Runtime integration and dispatch
```cpp
        // calculate s_i <- Q @ K
        at::native::cpublas::brgemm(
            /* M     */ h_size,
            /* N     */ n_size,
            /* K     */ head_size,
            /* lda   */ q_strideH,
            /* ldb   */ BLOCK_N,
            /* ldc   */ BLOCK_N,
            /* add_C */ false,
            /* A     */ q_ptr,
            /* B     */ Btmp0,
            /* C     */ s_i);

        const Vec scale_vec = Vec(sm_scale);
        for (int64_t h = 0; h < h_size; ++h) {
          // s_i <- s_i * scale
          at::vec::map<float>(
              [scale_vec](Vec x) { return x * scale_vec; }, s_i + h * BLOCK_N, s_i + h * BLOCK_N, n_size);

          // m_i: max value per row
          float m_i = at::vec::reduce_all<float>(
              [](Vec& x, Vec& y) { return at::vec::maximum(x, y); }, s_i + h * BLOCK_N, n_size);
          m_i = std::max(m_i, m_prime[h]);

          // m_delta <- exp(m' - m_i)
          m_delta[h] = std::exp(m_prime[h] - m_i);
```
**EN:** This section uses `brgemm`, `Vec`, `maximum` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`brgemm`、`Vec`、`maximum`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1308-1330: Runtime integration and dispatch
```cpp
          // s_delta <- exp(s_i - m_i)
          at::vec::map<float>(
              [m_i](Vec x) { return (x - Vec(m_i)).exp_u20(); }, s_delta + h * BLOCK_N, s_i + h * BLOCK_N, n_size);

          // s' <- s' * m_delta + sum(s_delta)
          s_prime[h] *= m_delta[h];
          s_prime[h] += at::vec::reduce_all<float>([](Vec& x, Vec& y) { return x + y; }, s_delta + h * BLOCK_N, n_size);

          m_prime[h] = m_i;

          // v' <- v' * m_delta
          float scale_m = m_delta[h];
          at::vec::map<float>(
              [scale_m](Vec x) { return x * Vec(scale_m); },
              v_prime + h * l_stride1,
              v_prime + h * l_stride1,
              head_size_v);

          // pad s_delta with 0 first and then convert to scalar_t
          fill_stub(s_delta + h * BLOCK_N + n_size, 0.f, padded_n_size - n_size);
          copy_stub<scalar_t, BLOCK_N>(s_delta2 + h * BLOCK_N, s_delta + h * BLOCK_N);
        }
```
**EN:** This section uses `Vec`, `fill_stub` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`Vec`、`fill_stub`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1331-1357: Runtime integration and dispatch
```cpp
        // calculate V' <- s_delta @ V + V'
        at::native::cpublas::brgemm(
            /* M     */ h_size,
            /* N     */ head_size_v,
            /* K     */ padded_n_size,  // n_size
            /* lda   */ BLOCK_N,
            /* ldb   */ head_size_v,
            /* ldc   */ l_stride1,
            /* add_C */ true,
            /* A     */ s_delta2,
            /* B     */ Btmp1,
            /* C     */ v_prime);
      }  // loop with KV blocks

      // only update v' when kv_split_size > 0
      if (kv_end > kv_start) {
        for (int64_t h = 0; h < h_size; ++h) {
          float s = 1 / s_prime[h];
          at::vec::map<float>(
              [s](Vec out) { return out * Vec(s); }, v_prime + h * l_stride1, v_prime + h * l_stride1, head_size_v);
          (v_prime + h * l_stride1)[head_size_v] = m_prime[h] + std::log(s_prime[h]);
        }
      }

      // move to the next index
      data_index_step(bs, batches, block_id, num_blocks, kv_id, num_kv_splits);
    }
```
**EN:** This section uses `brgemm`, `Vec`, `log` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`brgemm`、`Vec`、`log`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1358-1385: Runtime integration and dispatch
```cpp
    at::native::cpublas::brgemm_release();
  });

  decode_accumulate_kv_splits(
      output, attn_logits, batches, num_heads, head_size_v, num_kv_splits, l_stride1, l_stride2);
}  // MLA

template <typename scalar_t, typename index_t, int64_t BLOCK_N>
void decode_attention_grouped_kernel_impl(
    scalar_t* __restrict__ output,
    float* __restrict__ attn_logits,
    const scalar_t* __restrict__ query,
    const scalar_t* __restrict__ k_buffer,
    const scalar_t* __restrict__ v_buffer,
    const index_t* __restrict__ req_to_token,
    const int64_t* __restrict__ req_pool_indices,
    const int64_t* __restrict__ seq_lens,
    int64_t batches,
    int64_t num_heads,
    int64_t num_heads_kv,
    int64_t head_size,
    int64_t head_size_v,
    int64_t num_kv_splits,
    int64_t q_strideM,
    int64_t q_strideH,
    int64_t k_strideN,
    int64_t k_strideH,
    int64_t v_strideN,
```
**EN:** This section uses `brgemm_release`, `decode_accumulate_kv_splits` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`brgemm_release`、`decode_accumulate_kv_splits`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1386-1411: Runtime integration and dispatch
```cpp
    int64_t v_strideH,
    float sm_scale,
    float logit_cap,
    int64_t max_num_reqs,
    int64_t max_context_len,
    int64_t max_total_num_tokens) {
  using Vec = at::vec::Vectorized<float>;

  // block length for heads
  // we parallel on [batches, divup(num_heads, BLOCK_H), num_kv_splits]
  // use smaller BLOCK_H when batches is small to utilize all cores
  constexpr int64_t kBLOCK_H = 16;
  const int64_t BLOCK_H = std::min(4 * batches, kBLOCK_H);

  // strides
  const int64_t l_stride0 = num_heads * num_kv_splits * (head_size_v + 1);
  const int64_t l_stride1 = num_kv_splits * (head_size_v + 1);
  const int64_t l_stride2 = head_size_v + 1;

  const bool has_logit_cap = logit_cap > 0;
  float rlogit_cap = has_logit_cap ? 1 / logit_cap : 0.f;

  // partition the heads into blocks for parallel
  const int64_t num_groups = num_heads / num_heads_kv;
  const int64_t num_blocks = div_up(num_groups, BLOCK_H);
```
**EN:** This section uses `min`, `div_up`, `Vec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`min`、`div_up`、`Vec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1412-1440: Runtime integration and dispatch
```cpp
  // parallel on [batches, num_heads_kv, num_blocks, num_kv_splits]
  at::parallel_for(0, batches * num_heads_kv * num_blocks * num_kv_splits, 0, [&](int64_t begin, int64_t end) {
    int64_t bs{0}, head_kv_id{0}, block_id{0}, kv_id{0};
    data_index_init(begin, bs, batches, head_kv_id, num_heads_kv, block_id, num_blocks, kv_id, num_kv_splits);

    alignas(64) float s_i[BLOCK_H * BLOCK_N];
    float* __restrict__ s_delta = s_i;

    alignas(64) float s_prime[BLOCK_H];
    alignas(64) float m_prime[BLOCK_H];
    alignas(64) float m_delta[BLOCK_H];

    for (int64_t i = begin; i < end; ++i) {
      const int64_t h_start = head_kv_id * num_groups + block_id * BLOCK_H;
      const int64_t h_end = head_kv_id * num_groups + std::min(block_id * BLOCK_H + BLOCK_H, num_groups);
      const int64_t h_size = h_end - h_start;

      // get query
      const scalar_t* __restrict__ q_ptr = query + bs * q_strideM + h_start * q_strideH;

      int64_t seq_len_kv = seq_lens[bs];
      int64_t req_pool_id = req_pool_indices[bs];
      TORCH_CHECK(seq_len_kv <= max_context_len, "seq_len_kv out of scope!");
      TORCH_CHECK(req_pool_id < max_num_reqs, "req_pool_id out of scope!");

      const int64_t SPLIT_SIZE = div_up(seq_len_kv, num_kv_splits);
      const int64_t kv_start = kv_id * SPLIT_SIZE;
      const int64_t kv_end = std::min(kv_start + SPLIT_SIZE, seq_len_kv);
```
**EN:** This section uses `at::parallel_for`, `data_index_init`, `min` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`data_index_init`、`min`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1441-1468: Control flow and branching
```cpp
      fill_stub(s_prime, 0.f, BLOCK_H);
      fill_stub(m_prime, -std::numeric_limits<float>::infinity(), BLOCK_H);

      // get v_prime, and init to zero
      float* __restrict__ v_prime = attn_logits + bs * l_stride0 + h_start * l_stride1 + kv_id * l_stride2;
      for (int64_t h = 0; h < h_size; ++h) {
        fill_stub(v_prime + h * l_stride1, 0.f, head_size_v);
      }

      // loop over K and V sequence with BLOCK_N
      for (int64_t n = kv_start; n < kv_end; n += BLOCK_N) {
        int64_t n_size = std::min(BLOCK_N, kv_end - n);

        // calculate Q @ K
        index_gemm_kernel_nt<scalar_t, index_t>(
            /* A   */ q_ptr,
            /* B   */ k_buffer + head_kv_id * k_strideH,
            /* C   */ s_i,
            /* ind */ req_to_token + req_pool_id * max_context_len + n,
            /* scl */ sm_scale,
            /* M   */ h_size,
            /* N   */ n_size,
            /* K   */ head_size,
            /* lda */ q_strideH,
            /* ldb */ k_strideN,
            /* ldc */ BLOCK_N,
            /* mtt */ max_total_num_tokens);
```
**EN:** This section drives `fill_stub`, `min` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`fill_stub`、`min`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1469-1496: Runtime integration and dispatch
```cpp
        if (has_logit_cap) {
          at::vec::map<float>(
              [logit_cap, rlogit_cap](Vec x) { return Vec(logit_cap) * (x * Vec(rlogit_cap)).tanh(); },
              s_i,
              s_i,
              BLOCK_H * BLOCK_N);
        }

        // update the sm_scale coefficients
        for (int64_t h = 0; h < h_size; ++h) {
          // m_i: max value per row
          float m_i = at::vec::reduce_all<float>(
              [](Vec& x, Vec& y) { return at::vec::maximum(x, y); }, s_i + h * BLOCK_N, n_size);
          m_i = std::max(m_i, m_prime[h]);

          // m_delta <- exp(m' - m_i)
          m_delta[h] = std::exp(m_prime[h] - m_i);

          // s_delta <- exp(s_i - m_i)
          at::vec::map<float>(
              [m_i](Vec x) { return (x - Vec(m_i)).exp_u20(); }, s_delta + h * BLOCK_N, s_i + h * BLOCK_N, n_size);

          // s' <- s' * m_delta + sum(s_delta)
          s_prime[h] *= m_delta[h];
          s_prime[h] += at::vec::reduce_all<float>([](Vec& x, Vec& y) { return x + y; }, s_delta + h * BLOCK_N, n_size);

          m_prime[h] = m_i;
        }
```
**EN:** This section uses `Vec`, `maximum`, `max` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`Vec`、`maximum`、`max`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1497-1523: Runtime integration and dispatch
```cpp

        // calculate V' <- s_delta @ V + V' * m_delta
        index_gemm_kernel_nn<scalar_t, index_t>(
            /* A   */ s_delta,
            /* B   */ v_buffer + head_kv_id * v_strideH,
            /* C   */ v_prime,
            /* ind */ req_to_token + req_pool_id * max_context_len + n,
            /* scl */ m_delta,
            /* M   */ h_size,
            /* N   */ head_size_v,
            /* K   */ n_size,
            /* lda */ BLOCK_N,
            /* ldb */ v_strideN,
            /* ldc */ l_stride1,
            /* mtt */ max_total_num_tokens);
      }  // loop with KV blocks

      // only update v' when kv_split_size > 0
      if (kv_end > kv_start) {
        for (int64_t h = 0; h < h_size; ++h) {
          float s = 1 / s_prime[h];
          at::vec::map<float>(
              [s](Vec out) { return out * Vec(s); }, v_prime + h * l_stride1, v_prime + h * l_stride1, head_size_v);
          (v_prime + h * l_stride1)[head_size_v] = m_prime[h] + std::log(s_prime[h]);
        }
      }
```
**EN:** This section uses `Vec`, `log` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`Vec`、`log`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1524-1551: Runtime integration and dispatch
```cpp
      // move to the next index
      data_index_step(bs, batches, head_kv_id, num_heads_kv, block_id, num_blocks, kv_id, num_kv_splits);
    }
  });

  decode_accumulate_kv_splits(
      output, attn_logits, batches, num_heads, head_size_v, num_kv_splits, l_stride1, l_stride2);
}  // GQA/MQA

}  // anonymous namespace

// query:            [num_tokens, num_heads, head_size]
// output:           [num_tokens, num_heads, head_size]
// k_buffer:         [max_total_num_tokens, num_heads, head_size]
// v_buffer:         [max_total_num_tokens, num_heads, head_size_v]
// attn_logits:      [num_seqs, num_heads, num_kv_splits, head_size_v + 1]
// req_to_token:     [max_num_reqs, max_context_len] int32 or int64
// req_pool_indices: [num_seqs] int64
// seq_lens:         [num_seqs] int64
//
void decode_attention_cpu(
    at::Tensor& query,
    at::Tensor& k_buffer,
    at::Tensor& v_buffer,
    at::Tensor& output,
    at::Tensor& key,
    at::Tensor& value,
    at::Tensor& loc,
```
**EN:** This section uses `data_index_step`, `decode_accumulate_kv_splits` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`data_index_step`、`decode_accumulate_kv_splits`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1552-1580: Runtime integration and dispatch
```cpp
    at::Tensor& attn_logits,
    at::Tensor& req_to_token,
    at::Tensor& req_pool_indices,
    at::Tensor& seq_lens,
    double sm_scale,
    double logit_cap) {
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(query);
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(k_buffer);
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(v_buffer);
  // for MLA, key and value shares the same storage and value could be non-contiguous
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(key);
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(value);
  CHECK_DIM(3, query);
  CHECK_DIM(3, k_buffer);
  CHECK_DIM(3, v_buffer);
  CHECK_DIM(3, key);
  CHECK_DIM(3, value);
  CHECK_DIM(1, loc);

  int64_t num_seqs = seq_lens.size(0);
  int64_t max_num_reqs = req_to_token.size(0);
  int64_t max_context_len = req_to_token.size(1);
  int64_t max_total_num_tokens = k_buffer.size(0);

  int64_t num_heads = query.size(1);
  int64_t num_heads_kv = k_buffer.size(1);
  int64_t head_size = query.size(2);
  int64_t head_size_v = v_buffer.size(2);
```
**EN:** This section uses `CHECK_LAST_DIM_CONTIGUOUS_INPUT`, `CHECK_DIM` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_LAST_DIM_CONTIGUOUS_INPUT`、`CHECK_DIM`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1581-1603: Runtime integration and dispatch
```cpp
  int64_t num_kv_splits = attn_logits.size(2);

  CHECK_EQ(loc.numel(), num_seqs);
  CHECK_EQ(attn_logits.size(0), num_seqs);
  CHECK_EQ(attn_logits.size(1), num_heads);
  CHECK_EQ(attn_logits.size(3), head_size_v + 1);
  CHECK_EQ(attn_logits.scalar_type(), at::kFloat);

  // strides for query
  int64_t q_strideM = query.stride(0);
  int64_t q_strideH = query.stride(1);

  // strides for k_buffer and v_buffer
  int64_t k_strideN = k_buffer.stride(0);
  int64_t k_strideH = k_buffer.stride(1);
  int64_t v_strideN = v_buffer.stride(0);
  int64_t v_strideH = v_buffer.stride(1);
  // strides for new key and value
  int64_t nk_strideN = key.stride(0);
  int64_t nk_strideH = key.stride(1);
  int64_t nv_strideN = value.stride(0);
  int64_t nv_strideH = value.stride(1);
```
**EN:** This section uses `CHECK_EQ`, `stride` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_EQ`、`stride`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1604-1628: Runtime integration and dispatch
```cpp
  // check index data types
  const auto index_dtype = req_to_token.scalar_type();
  TORCH_CHECK(
      index_dtype == at::kInt || index_dtype == at::kLong,
      "decode: expect req_to_token to be int32 or int64, got ",
      index_dtype);
  TORCH_CHECK(seq_lens.scalar_type() == at::kLong, "decode: expect req_lens to be int64, got ", seq_lens.scalar_type());
  TORCH_CHECK(
      req_pool_indices.scalar_type() == at::kLong,
      "decode: expect req_pool_indices to be int64, got ",
      req_pool_indices.scalar_type());

  // check if we have MLA here
  void* k_buffer_data = k_buffer.data_ptr();
  void* v_buffer_data = v_buffer.data_ptr();
  const bool is_mla = (k_buffer_data == v_buffer_data) && (num_heads_kv == 1) && (head_size == head_size_v + 64);

  // block length for k_buffer and v_buffer
  constexpr int BLOCK_N = 256;

  // buffer for packing k_cache and v_cache
  int num_threads = at::get_num_threads();
  int64_t size_per_thread = is_mla ? BLOCK_N * head_size + BLOCK_N * head_size_v : 0;
  auto buffer = at::empty({num_threads, size_per_thread}, k_buffer.options());
```
**EN:** This section uses `scalar_type`, `TORCH_CHECK`, `data_ptr` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`scalar_type`、`TORCH_CHECK`、`data_ptr`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 1629-1651: Local implementation details
```cpp
  AT_DISPATCH_REDUCED_FLOATING_TYPES(query.scalar_type(), "decode_attention_kernel", [&] {
    AT_DISPATCH_INDEX_TYPES(index_dtype, "decode_attention_indices", [&] {
      // update the kv buffer
      decode_set_kv_buffer(
          (scalar_t*)k_buffer_data,
          (scalar_t*)v_buffer_data,
          key.data_ptr<scalar_t>(),
          value.data_ptr<scalar_t>(),
          loc.data_ptr<int64_t>(),
          num_seqs,
          num_heads_kv,
          head_size,
          head_size_v,
          k_strideN,
          k_strideH,
          v_strideN,
          v_strideH,
          nk_strideN,
          nk_strideH,
          nv_strideN,
          nv_strideH,
          is_mla);
```
**EN:** This section fills in the local implementation details around `decode_set_kv_buffer`, completing the behavior required by the file.
**CN:** 本段补充了`decode_set_kv_buffer`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 1652-1679: Control flow and branching
```cpp
      if (num_heads == num_heads_kv) {
        // MHA
        decode_attention_kernel_impl<scalar_t, index_t, BLOCK_N>(
            output.data_ptr<scalar_t>(),
            attn_logits.data_ptr<float>(),
            query.data_ptr<scalar_t>(),
            (const scalar_t*)k_buffer_data,
            (const scalar_t*)v_buffer_data,
            req_to_token.data_ptr<index_t>(),
            req_pool_indices.data_ptr<int64_t>(),
            seq_lens.data_ptr<int64_t>(),
            num_seqs,
            num_heads,
            head_size,
            head_size_v,
            num_kv_splits,
            q_strideM,
            q_strideH,
            k_strideN,
            k_strideH,
            v_strideN,
            v_strideH,
            sm_scale,
            logit_cap,
            max_num_reqs,
            max_context_len,
            max_total_num_tokens);
      } else if (is_mla) {
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1680-1707: Local implementation details
```cpp
        // MLA
        decode_attention_mla_kernel_impl<scalar_t, index_t, BLOCK_N>(
            output.data_ptr<scalar_t>(),
            attn_logits.data_ptr<float>(),
            query.data_ptr<scalar_t>(),
            (const scalar_t*)k_buffer_data,
            (const scalar_t*)v_buffer_data,
            req_to_token.data_ptr<index_t>(),
            req_pool_indices.data_ptr<int64_t>(),
            seq_lens.data_ptr<int64_t>(),
            buffer.data_ptr<scalar_t>(),
            num_seqs,
            num_heads,
            head_size,
            head_size_v,
            num_kv_splits,
            q_strideM,
            q_strideH,
            k_strideN,
            k_strideH,
            v_strideN,
            v_strideH,
            sm_scale,
            logit_cap,
            max_num_reqs,
            max_context_len,
            max_total_num_tokens,
            size_per_thread);
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 1708-1736: Local implementation details
```cpp
      } else {
        // GQA/MQA
        decode_attention_grouped_kernel_impl<scalar_t, index_t, BLOCK_N>(
            output.data_ptr<scalar_t>(),
            attn_logits.data_ptr<float>(),
            query.data_ptr<scalar_t>(),
            (const scalar_t*)k_buffer_data,
            (const scalar_t*)v_buffer_data,
            req_to_token.data_ptr<index_t>(),
            req_pool_indices.data_ptr<int64_t>(),
            seq_lens.data_ptr<int64_t>(),
            num_seqs,
            num_heads,
            num_heads_kv,
            head_size,
            head_size_v,
            num_kv_splits,
            q_strideM,
            q_strideH,
            k_strideN,
            k_strideH,
            v_strideN,
            v_strideH,
            sm_scale,
            logit_cap,
            max_num_reqs,
            max_context_len,
            max_total_num_tokens);
      }
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 1737-1739: Local implementation details
```cpp
    });
  });
}
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `common.h`, `gemm.h`, `vec.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cpu / decode.cpp
