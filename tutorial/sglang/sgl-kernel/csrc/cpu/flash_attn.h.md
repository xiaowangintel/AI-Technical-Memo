# flash_attn.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/flash_attn.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Runtime integration and dispatch
```cpp
#pragma once
#include "common.h"
#include "vec.h"
#include "vec_pack.h"

template <typename scalar_t>
inline void fill_stub(scalar_t* __restrict__ out, float val, int size) {
  using Vec = at::vec::Vectorized<scalar_t>;
  constexpr int kVecSize = Vec::size();
  const Vec data_vec = Vec(static_cast<scalar_t>(val));
  int d = 0;
#pragma GCC unroll 4
  for (; d <= size - kVecSize; d += kVecSize) {
    data_vec.store(out + d);
  }
  if (size - d > 0) {
    data_vec.store(out + d, size - d);
  }
}
```
**EN:** This section uses `fill_stub`, `Vec`, `store` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fill_stub`、`Vec`、`store`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 21-40: Runtime integration and dispatch
```cpp
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
  };
  Unroll<COLS>{}(store);
}
```
**EN:** This section uses `copy_stub`, `static_assert`, `loadu` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`copy_stub`、`static_assert`、`loadu`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 41-59: Runtime integration and dispatch
```cpp
template <typename scalar_t>
inline void copy_stub(scalar_t* __restrict__ out, const float* __restrict__ acc, float s, int size) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;
  constexpr int kVecSize = bVec::size();
  const fVec s_fvec = fVec(s);
  int d = 0;
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
**EN:** This section uses `copy_stub`, `fVec`, `store` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`copy_stub`、`fVec`、`store`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 60-86: Runtime integration and dispatch
```cpp
#if defined(CPU_CAPABILITY_AVX512)
template <>
inline void copy_stub<at::BFloat16>(at::BFloat16* __restrict__ out, const float* __restrict__ acc, float s, int size) {
  const __m512 vscale = _mm512_set1_ps(s);
  int d = 0;
#pragma GCC unroll 4
  for (; d <= size - 32; d += 32) {
    __m512 va0 = _mm512_mul_ps(_mm512_loadu_ps(acc + d), vscale);
    __m512 va1 = _mm512_mul_ps(_mm512_loadu_ps(acc + d + 16), vscale);
    __m512i vb = (__m512i)(_mm512_cvtne2ps_pbh(va1, va0));
    _mm512_storeu_si512(out + d, vb);
  }
  int remainder = size - d;
  if (remainder > 0) {
    if (remainder <= 16) {
      const __mmask16 vmask = (1ULL << remainder) - 1;
      __m512 va = _mm512_mul_ps(_mm512_maskz_loadu_ps(vmask, acc + d), vscale);
      __m256i vb = (__m256i)(_mm512_cvtneps_pbh(va));
      _mm256_mask_storeu_epi16(reinterpret_cast<__m256i*>(out + d), vmask, vb);
    } else {  // remainder > 16
      const __mmask16 vmask = (1ULL << (remainder - 16)) - 1;
      __m512 va0 = _mm512_mul_ps(_mm512_loadu_ps(acc + d), vscale);
      __m512 va1 = _mm512_mul_ps(_mm512_maskz_loadu_ps(vmask, acc + d + 16), vscale);
      __m512i vb = (__m512i)(_mm512_cvtne2ps_pbh(va1, va0));
      const __mmask32 vmask2 = (1ULL << remainder) - 1;
      _mm512_mask_storeu_epi16(reinterpret_cast<__m512i*>(out + d), vmask2, vb);
    }
```
**EN:** This section uses `defined`, `_mm512_set1_ps`, `_mm512_mul_ps` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`defined`、`_mm512_set1_ps`、`_mm512_mul_ps`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 87-111: Types and data layout
```cpp
  }
}
#endif

template <typename scalar_t, int BLOCK_M, int BLOCK_N>
struct flash_attn_softmax {
  static inline void apply(
      float* __restrict__ s_i,
      scalar_t* __restrict__ s_delta2,
      float* __restrict__ v_prime,
      float* __restrict__ s_prime,
      float* __restrict__ m_prime,
      int m_size,
      int n_size,
      int padded_n_size,
      int head_size_v,
      const float sm_scale) {
    using Vec = at::vec::Vectorized<float>;
    const Vec scale_vec = Vec(sm_scale);
    float* s_delta = s_i;
    for (int row = 0; row < m_size; ++row) {
      // s_i <- s_i * scale
      at::vec::map<float>(
          [scale_vec](Vec x) { return x * scale_vec; }, s_i + row * BLOCK_N, s_i + row * BLOCK_N, n_size);
```
**EN:** This section defines `flash_attn_softmax`, `apply`, `Vec`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`flash_attn_softmax`、`apply`、`Vec`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 112-136: Runtime integration and dispatch
```cpp
      // m_i: max value per row
      float m_i = at::vec::reduce_all<float>(
          [](Vec& x, Vec& y) { return at::vec::maximum(x, y); }, s_i + row * BLOCK_N, n_size);
      m_i = std::max(m_i, m_prime[row]);

      // m_delta <- exp(m' - m_i)
      float m_delta = std::exp(m_prime[row] - m_i);

      // s_delta <- exp(s_i - m_i)
      at::vec::map<float>(
          [m_i](Vec x) { return (x - Vec(m_i)).fexp_u20(); }, s_delta + row * BLOCK_N, s_i + row * BLOCK_N, n_size);

      // s' <- s' * m_delta + sum(s_delta)
      s_prime[row] *= m_delta;
      s_prime[row] += at::vec::reduce_all<float>([](Vec& x, Vec& y) { return x + y; }, s_delta + row * BLOCK_N, n_size);

      m_prime[row] = m_i;

      // v' <- v' * m_delta
      at::vec::map<float>(
          [m_delta](Vec x) { return x * Vec(m_delta); },
          v_prime + row * head_size_v,
          v_prime + row * head_size_v,
          head_size_v);
```
**EN:** This section uses `maximum`, `max`, `exp` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`maximum`、`max`、`exp`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 137-161: Types and data layout
```cpp
      // Keep s_delta row-major for the following brgemm(P @ V), and only
      // convert the columns that brgemm will consume.
      fill_stub(s_delta + row * BLOCK_N + n_size, 0.f, padded_n_size - n_size);
      copy_stub<scalar_t>(s_delta2 + row * BLOCK_N, s_delta + row * BLOCK_N, 1.f, padded_n_size);
    }
  }
};

#if defined(CPU_CAPABILITY_AVX512)
template <int BLOCK_M, int BLOCK_N>
struct flash_attn_softmax<at::BFloat16, BLOCK_M, BLOCK_N> {
  static inline void apply(
      float* __restrict__ s_i,
      at::BFloat16* __restrict__ s_delta2,
      float* __restrict__ v_prime,
      float* __restrict__ s_prime,
      float* __restrict__ m_prime,
      int m_size,
      int n_size,
      int padded_n_size,
      int head_size_v,
      const float sm_scale) {
    float* s_delta = s_i;
    const __m512 vscale = _mm512_set1_ps(sm_scale);
```
**EN:** This section defines `flash_attn_softmax`, `apply`, `fill_stub`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`flash_attn_softmax`、`apply`、`fill_stub`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 162-186: Templates, aliases, and constants
```cpp
    int n_remainder = n_size & 15;  // 0xF
    const __mmask16 vmask = (1ULL << n_remainder) - 1;

    int v_remainder = head_size_v & 15;  // 0xF
    const __mmask16 vmask1 = (1ULL << v_remainder) - 1;

    constexpr float NEG_INF = -std::numeric_limits<float>::infinity();

    __m512 va;
    __m256i vb;
    __m512 vmax;
    __m512 vsum;
    __m512 vmdelta;

    const __m512 vneg_inf = _mm512_set1_ps(NEG_INF);

    for (int m = 0; m < m_size; ++m) {
      vmax = vneg_inf;

      // s_i <- s_i * scale
      int n = 0;
      for (; n <= n_size - 16; n += 16) {
        va = _mm512_mul_ps(_mm512_loadu_ps(s_i + m * BLOCK_N + n), vscale);
        vmax = _mm512_max_ps(va, vmax);
      }
```
**EN:** This section defines `infinity`, `_mm512_set1_ps`, `_mm512_mul_ps`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`infinity`、`_mm512_set1_ps`、`_mm512_mul_ps`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 187-209: Control flow and branching
```cpp
      if (n_remainder > 0) {
        va = _mm512_mul_ps(_mm512_mask_loadu_ps(vneg_inf, vmask, s_i + m * BLOCK_N + n), vscale);
        vmax = _mm512_max_ps(va, vmax);
      }

      // m_i: max value per row
      float m_i = _mm512_reduce_max_ps(vmax);
      m_i = std::max(m_i, m_prime[m]);
      vmax = _mm512_set1_ps(m_i);

      // m_delta <- exp(m' - m_i)
      float m_delta = std::exp(m_prime[m] - m_i);

      // s_delta <- exp(s_i - m_i)
      vsum = _mm512_setzero_ps();
      for (n = 0; n <= n_size - 16; n += 16) {
        va = _mm512_mul_ps(_mm512_loadu_ps(s_i + m * BLOCK_N + n), vscale);
        va = _mm512_fexp_u20_ps(_mm512_sub_ps(va, vmax));
        vsum = _mm512_add_ps(vsum, va);

        vb = (__m256i)(_mm512_cvtneps_pbh(va));
        _mm256_storeu_si256(reinterpret_cast<__m256i*>(s_delta2 + m * BLOCK_N + n), vb);
      }
```
**EN:** This section drives `_mm512_mul_ps`, `_mm512_max_ps`, `_mm512_reduce_max_ps` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`_mm512_mul_ps`、`_mm512_max_ps`、`_mm512_reduce_max_ps`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 210-231: Control flow and branching
```cpp
      if (n_remainder > 0) {
        va = _mm512_mul_ps(_mm512_mask_loadu_ps(vneg_inf, vmask, s_i + m * BLOCK_N + n), vscale);
        va = _mm512_fexp_u20_ps(_mm512_sub_ps(va, vmax));
        vsum = _mm512_add_ps(vsum, va);

        vb = (__m256i)(_mm512_cvtneps_pbh(va));
        _mm256_mask_storeu_epi16(reinterpret_cast<__m256i*>(s_delta2 + m * BLOCK_N + n), vmask, vb);
      }

      // s' <- s' * m_delta + sum(s_delta)
      s_prime[m] *= m_delta;
      s_prime[m] += _mm512_reduce_add_ps(vsum);

      m_prime[m] = m_i;

      // pad s_delta with 0, pad_size range from [0, 32)
      int pad_size = padded_n_size - n_size;
      if (pad_size > 0) {
        const __m512i vzero = _mm512_setzero_si512();
        __mmask32 vmask2 = (1ULL << pad_size) - 1;
        _mm512_mask_storeu_epi16(reinterpret_cast<__m512i*>(s_delta2 + m * BLOCK_N + n_size), vmask2, vzero);
      }
```
**EN:** This section drives `_mm512_mul_ps`, `_mm512_fexp_u20_ps`, `_mm512_add_ps` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`_mm512_mul_ps`、`_mm512_fexp_u20_ps`、`_mm512_add_ps`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 232-246: Control flow and branching
```cpp

      // v' <- v' * m_delta
      vmdelta = _mm512_set1_ps(m_delta);
      int k = 0;
      for (; k <= head_size_v - 16; k += 16) {
        va = _mm512_mul_ps(_mm512_loadu_ps(v_prime + m * head_size_v + k), vmdelta);
        _mm512_storeu_ps(reinterpret_cast<__m512*>(v_prime + m * head_size_v + k), va);
      }
      if (v_remainder > 0) {
        va = _mm512_mul_ps(_mm512_maskz_loadu_ps(vmask1, v_prime + m * head_size_v + k), vmdelta);
        _mm512_mask_storeu_ps(reinterpret_cast<__m512*>(v_prime + m * head_size_v + k), vmask1, va);
      }
    }
  }
};
```
**EN:** This section drives `_mm512_set1_ps`, `_mm512_mul_ps`, `_mm512_storeu_ps` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`_mm512_set1_ps`、`_mm512_mul_ps`、`_mm512_storeu_ps`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 247-247: Local implementation details
```cpp
#endif
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `common.h`, `vec.h`, `vec_pack.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cpu / flash_attn.h
