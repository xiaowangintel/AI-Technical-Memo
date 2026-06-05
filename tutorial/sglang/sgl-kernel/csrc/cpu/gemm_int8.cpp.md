# gemm_int8.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/gemm_int8.cpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Namespace and shared declarations
```cpp
#include "common.h"
#include "gemm.h"
#include "vec.h"

namespace {

template <typename scalar_t, bool has_bias, int BLOCK_N>
struct scale_C {
  static inline void apply(
      scalar_t* __restrict__ C,
      const int32_t* __restrict__ Ctmp,
      const int32_t* __restrict__ Bcomp,
      const float* __restrict__ bias,
      float As,
      const float* __restrict__ Bs) {
    TORCH_CHECK(false, "scale_C: scalar path not implemented!");
  }
};
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 20-46: Types and data layout
```cpp
#if defined(CPU_CAPABILITY_AVX512)
template <bool has_bias, int BLOCK_N>
struct scale_C<at::BFloat16, has_bias, BLOCK_N> {
  static inline void apply(
      at::BFloat16* __restrict__ C,
      const int32_t* __restrict__ Ctmp,
      const int32_t* __restrict__ Bcomp,
      const float* __restrict__ bias,
      float As,
      const float* __restrict__ Bs) {
    constexpr int COLS = BLOCK_N / 16;
    static_assert(COLS % 2 == 0);

    __m512 vc[COLS];
    __m512 vd0 = _mm512_set1_ps(As);

    auto compute = [&](auto col) {
      __m512 vd1 = _mm512_loadu_ps(Bs + col * 16);
      __m512i vcomp = _mm512_loadu_si512(Bcomp + col * 16);
      __m512i vc32 = _mm512_loadu_si512(Ctmp + col * 16);
      vc[col] = _mm512_cvtepi32_ps(_mm512_sub_epi32(vc32, vcomp));
      if constexpr (has_bias) {
        __m512 vbias = _mm512_loadu_ps(bias + col * 16);
        vc[col] = _mm512_fmadd_ps(_mm512_mul_ps(vc[col], vd0), vd1, vbias);
      } else {
        vc[col] = _mm512_mul_ps(_mm512_mul_ps(vc[col], vd0), vd1);
      }
```
**EN:** This section defines `scale_C`, `apply`, `static_assert`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`scale_C`、`apply`、`static_assert`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 47-70: Types and data layout
```cpp
    };
    Unroll<COLS>{}(compute);

    auto storec = [&](auto col) {
      // for COLS = 2, 4 use 512bit store
      if constexpr (col % 2 == 0) {
        _mm512_storeu_si512(
            reinterpret_cast<__m512i*>((C + col * 16)), (__m512i)(_mm512_cvtne2ps_pbh(vc[col + 1], vc[col + 0])));
      }
    };
    Unroll<COLS>{}(storec);
  }
};
#endif

template <typename scalar_t, bool has_bias, int BLOCK_M, int BLOCK_N>
struct tinygemm_kernel_nn {
  static inline void apply(
      const uint8_t* __restrict__ A,
      const int8_t* __restrict__ B,
      scalar_t* __restrict__ C,
      const float* __restrict__ As,
      const float* __restrict__ Bs,
      const int32_t* __restrict__ Bcomp,
```
**EN:** This section defines `tinygemm_kernel_nn`, `_mm512_storeu_si512`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`tinygemm_kernel_nn`、`_mm512_storeu_si512`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 71-98: Types and data layout
```cpp
      const float* __restrict__ bias,
      int64_t K,
      int64_t lda,
      int64_t ldb,
      int64_t ldc) {
    TORCH_CHECK(false, "tinygemm_kernel_nn: scalar path not implemented!");
  }
};

#if defined(CPU_CAPABILITY_AVX512)
template <bool has_bias, int BLOCK_M, int BLOCK_N>
struct tinygemm_kernel_nn<at::BFloat16, has_bias, BLOCK_M, BLOCK_N> {
  static inline void apply(
      const uint8_t* __restrict__ A,
      const int8_t* __restrict__ B,
      at::BFloat16* __restrict__ C,
      const float* __restrict__ As,
      const float* __restrict__ Bs,
      const int32_t* __restrict__ Bcomp,
      const float* __restrict__ bias,
      int64_t K,
      int64_t lda,
      int64_t ldb,
      int64_t ldc) {
    constexpr int ROWS = BLOCK_M;
    constexpr int COLS = BLOCK_N / 16;
    static_assert(COLS % 2 == 0);
```
**EN:** This section defines `tinygemm_kernel_nn`, `apply`, `TORCH_CHECK`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`tinygemm_kernel_nn`、`apply`、`TORCH_CHECK`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 99-124: Templates, aliases, and constants
```cpp
    // prefetch distance
    constexpr int PREFETCH_SIZE_K = 0;

    __m512i va;
    __m512i vb[COLS];
    __m512i vc[ROWS * COLS];
    __m512i vcomp[COLS];
    __m512 vd0;
    __m512 vd1[COLS];

    // oops! 4x4 spills but we use 4x2
    __m512 vbias[COLS];

    // [NOTE]: s8s8 igemm compensation in avx512-vnni
    //
    // avx512-vnni has no s8s8, so we need to change s8s8 to u8s8 with compensate:
    //
    //   a * b = (a + 128) * b - 128 * b
    //   s   s       u       s    u    s
    //
    // 1) 128 * b is pre-computed when packing B to vnni formats
    // 2) a + 128 is fused when dynamically quantize A
    //
    auto loadc = [&](auto i) { vc[i] = _mm512_set1_epi32(0); };
    Unroll<ROWS * COLS>{}(loadc);
```
**EN:** This section defines `_mm512_set1_epi32`, `PREFETCH_SIZE_K`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`_mm512_set1_epi32`、`PREFETCH_SIZE_K`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 125-148: Templates, aliases, and constants
```cpp
    const int64_t K4 = K >> 2;
    const int64_t lda4 = lda >> 2;
    const int64_t ldb4 = ldb;  // ldb * 4 >> 2;
    const int32_t* a_ptr = reinterpret_cast<const int32_t*>(A);
    const int32_t* b_ptr = reinterpret_cast<const int32_t*>(B);

    auto compute = [&](auto i, int64_t k) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;

      if constexpr (col == 0) {
        va = _mm512_set1_epi32(a_ptr[row * lda4 + k]);
      }
      if constexpr (row == 0) {
        vb[col] = _mm512_loadu_si512(b_ptr + k * ldb4 + col * 16);
        if constexpr (PREFETCH_SIZE_K > 0) {
          _mm_prefetch(b_ptr + (k + PREFETCH_SIZE_K) * ldb4 + col * 16, _MM_HINT_T0);
        }
      }
      vc[i] = _mm512_dpbusd_epi32(vc[i], va, vb[col]);
    };
    for (int64_t k = 0; k < K4; ++k) {
      Unroll<ROWS * COLS>{}(compute, k);
    }
```
**EN:** This section defines `_mm512_set1_epi32`, `_mm512_loadu_si512`, `_mm_prefetch`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`_mm512_set1_epi32`、`_mm512_loadu_si512`、`_mm_prefetch`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 149-172: Templates, aliases, and constants
```cpp

    auto storec = [&](auto i) {
      constexpr int row = i / COLS;
      constexpr int col = i % COLS;

      // load a scale
      if constexpr (col == 0) {
        vd0 = _mm512_set1_ps(As[row]);
      }
      // load b scale and vcomp per 2 vectors
      // also load bias if any
      if constexpr (row == 0) {
        if constexpr (col % 2 == 0) {
          vd1[col + 0] = _mm512_loadu_ps(Bs + col * 16);
          vd1[col + 1] = _mm512_loadu_ps(Bs + col * 16 + 16);
          vcomp[col + 0] = _mm512_loadu_si512(Bcomp + col * 16);
          vcomp[col + 1] = _mm512_loadu_si512(Bcomp + col * 16 + 16);
          if constexpr (has_bias) {
            vbias[col + 0] = _mm512_loadu_ps(bias + col * 16);
            vbias[col + 1] = _mm512_loadu_ps(bias + col * 16 + 16);
          }
        }
      }
```
**EN:** This section defines `_mm512_set1_ps`, `_mm512_loadu_ps`, `_mm512_loadu_si512`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`_mm512_set1_ps`、`_mm512_loadu_ps`、`_mm512_loadu_si512`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 173-193: Templates, aliases, and constants
```cpp
      // for COLS = 2, 4 use 512bit store
      if constexpr (col % 2 == 0) {
        __m512 vc0 = _mm512_cvtepi32_ps(_mm512_sub_epi32(vc[row * COLS + col + 0], vcomp[col + 0]));
        __m512 vc1 = _mm512_cvtepi32_ps(_mm512_sub_epi32(vc[row * COLS + col + 1], vcomp[col + 1]));
        if constexpr (has_bias) {
          vc0 = _mm512_fmadd_ps(_mm512_mul_ps(vc0, vd0), vd1[col + 0], vbias[col + 0]);
          vc1 = _mm512_fmadd_ps(_mm512_mul_ps(vc1, vd0), vd1[col + 1], vbias[col + 1]);
        } else {
          vc0 = _mm512_mul_ps(_mm512_mul_ps(vc0, vd0), vd1[col + 0]);
          vc1 = _mm512_mul_ps(_mm512_mul_ps(vc1, vd0), vd1[col + 1]);
        }

        _mm512_storeu_si512(
            reinterpret_cast<__m512i*>((C + row * ldc + col * 16)), (__m512i)(_mm512_cvtne2ps_pbh(vc1, vc0)));
      }
    };
    Unroll<ROWS * COLS>{}(storec);
  }
};
#endif
```
**EN:** This section defines `_mm512_cvtepi32_ps`, `_mm512_fmadd_ps`, `_mm512_mul_ps`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`_mm512_cvtepi32_ps`、`_mm512_fmadd_ps`、`_mm512_mul_ps`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 194-217: Templates, aliases, and constants
```cpp
#define LAUNCH_TINYGEMM_KERNEL_NN(MB_SIZE, NB_SIZE)                \
  tinygemm_kernel_nn<scalar_t, has_bias, MB_SIZE, NB_SIZE>::apply( \
      A + mb_start * lda,                                          \
      B + nb_start * 4,                                            \
      C + mb_start * ldc + nb_start,                               \
      As + mb_start,                                               \
      Bs + nb_start,                                               \
      Bcomp + nb_start,                                            \
      has_bias ? bias + nb_start : nullptr,                        \
      K,                                                           \
      lda,                                                         \
      ldb,                                                         \
      ldc);

template <typename scalar_t, bool has_bias>
void tinygemm_kernel(
    const uint8_t* __restrict__ A,
    const int8_t* __restrict__ B,
    scalar_t* __restrict__ C,
    int32_t* __restrict__ Ctmp,
    const float* __restrict__ As,
    const float* __restrict__ Bs,
    const float* __restrict__ bias,
    int64_t M,
```
**EN:** This section defines `LAUNCH_TINYGEMM_KERNEL_NN`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`LAUNCH_TINYGEMM_KERNEL_NN`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 218-237: Runtime integration and dispatch
```cpp
    int64_t N,
    int64_t K,
    int64_t lda,
    int64_t ldb,
    int64_t ldc,
    bool brg) {
  // B compensation
  const int32_t* Bcomp = reinterpret_cast<const int32_t*>(B + block_size_n() * K);

  if (brg) {
    constexpr int BLOCK_N = block_size_n();
    at::native::cpublas::brgemm(M, N, K, lda, ldb, BLOCK_N, /* add_C */ false, A, B, Ctmp);

    // apply compensation and scale
    for (int64_t m = 0; m < M; ++m) {
      scale_C<scalar_t, has_bias, BLOCK_N>::apply(C + m * ldc, Ctmp + m * BLOCK_N, Bcomp, bias, As[m], Bs);
    }
    return;
  }
```
**EN:** This section uses `block_size_n`, `brgemm`, `apply` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`block_size_n`、`brgemm`、`apply`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 238-261: Templates, aliases, and constants
```cpp
  // pattern: 1-4-16
  constexpr int64_t BLOCK_M = 4;
  constexpr int64_t BLOCK_N = 64;
  const int64_t MB = div_up(M, BLOCK_M);
  const int64_t NB = div_up(N, BLOCK_N);
  for (int64_t mb = 0; mb < MB; ++mb) {
    int64_t mb_start = mb * BLOCK_M;
    int64_t mb_size = std::min(BLOCK_M, M - mb_start);
    for (int64_t nb = 0; nb < NB; ++nb) {
      int64_t nb_start = nb * BLOCK_N;
      int64_t nb_size = std::min(BLOCK_N, N - nb_start);

      switch (mb_size << 4 | nb_size >> 4) {
        // mb_size = 1
        case 0x12:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 32);
          break;
        case 0x14:
          LAUNCH_TINYGEMM_KERNEL_NN(1, 64);
          break;
        // mb_size = 2
        case 0x22:
          LAUNCH_TINYGEMM_KERNEL_NN(2, 32);
          break;
```
**EN:** This section defines `div_up`, `min`, `LAUNCH_TINYGEMM_KERNEL_NN`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`div_up`、`min`、`LAUNCH_TINYGEMM_KERNEL_NN`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 262-285: Runtime integration and dispatch
```cpp
        case 0x24:
          LAUNCH_TINYGEMM_KERNEL_NN(2, 64);
          break;
        // mb_size = 3
        case 0x32:
          LAUNCH_TINYGEMM_KERNEL_NN(3, 32);
          break;
        case 0x34:
          LAUNCH_TINYGEMM_KERNEL_NN(3, 64);
          break;
        // mb_size = 4
        case 0x42:
          LAUNCH_TINYGEMM_KERNEL_NN(4, 32);
          break;
        case 0x44:
          LAUNCH_TINYGEMM_KERNEL_NN(4, 64);
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

### Lines 286-311: Templates, aliases, and constants
```cpp
template <typename scalar_t>
void int8_scaled_mm_kernel_impl(
    scalar_t* __restrict__ out,
    const uint8_t* __restrict__ mat1,
    const int8_t* __restrict__ mat2,
    const float* __restrict__ scales1,
    const float* __restrict__ scales2,
    const float* __restrict__ bias,
    int64_t M,
    int64_t N,
    int64_t K) {
  constexpr int64_t BLOCK_M = block_size_m();
  constexpr int64_t BLOCK_N = block_size_n();
  const int64_t MB = div_up(M, BLOCK_M);
  const int64_t NB = div_up(N, BLOCK_N);

  const bool use_brgemm = can_use_brgemm<int8_t>(M);

  // K + 4 after compensation
  const int64_t packed_row_size = get_row_size<int8_t>(K);

  AT_DISPATCH_BOOL(bias != nullptr, has_bias, [&] {
    parallel_2d(MB, NB, [&](int64_t mb0, int64_t mb1, int64_t nb0, int64_t nb1) {
      // for brgemm, use int32_t for accumulate
      alignas(64) int32_t Ctmp[BLOCK_M * BLOCK_N];
```
**EN:** This section defines `int8_scaled_mm_kernel_impl`, `parallel_2d`, `block_size_m`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`int8_scaled_mm_kernel_impl`、`parallel_2d`、`block_size_m`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 312-334: Local implementation details
```cpp
      loop_2d<int8_t>(mb0, mb1, nb0, nb1, BLOCK_N * K, [&](int64_t mb, int64_t nb, int64_t nb_offset) {
        int mb_start = mb * BLOCK_M;
        int mb_size = std::min(M - mb_start, BLOCK_M);
        int nb_start = nb * BLOCK_N;
        int nb_size = std::min(N - nb_start, BLOCK_N);

        tinygemm_kernel<scalar_t, has_bias>(
            /*   A */ mat1 + mb_start * K,
            /*   B */ mat2 + nb_start * packed_row_size /* nb * BLOCK_N * (K + 4) */,
            /*   C */ out + mb_start * N + nb_start,
            /* Ctmp*/ Ctmp,
            /*  As */ scales1 + mb_start,
            /*  Bs */ scales2 + nb_start,
            /* bias*/ bias + nb_start,
            /*   M */ mb_size,
            /*   N */ nb_size,
            /*   K */ K,
            /* lda */ K,
            /* ldb */ nb_size,
            /* ldc */ N,
            /* brg */ use_brgemm);
      });
```
**EN:** This section fills in the local implementation details around `min`, completing the behavior required by the file.
**CN:** 本段补充了`min`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 335-361: Runtime integration and dispatch
```cpp
      if (use_brgemm) {
        at::native::cpublas::brgemm_release();
      }
    });
  });
}

}  // anonymous namespace

// tinygemm interface
template <typename scalar_t>
void tinygemm_kernel(
    const uint8_t* __restrict__ A,
    const int8_t* __restrict__ B,
    scalar_t* __restrict__ C,
    int32_t* __restrict__ Ctmp,
    const float* __restrict__ As,
    const float* __restrict__ Bs,
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t lda,
    int64_t ldb,
    int64_t ldc,
    bool brg) {
  tinygemm_kernel<scalar_t, false>(A, B, C, Ctmp, As, Bs, nullptr, M, N, K, lda, ldb, ldc, brg);
}
```
**EN:** This section uses `tinygemm_kernel`, `brgemm_release` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`tinygemm_kernel`、`brgemm_release`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 362-385: Runtime integration and dispatch
```cpp

#define INSTANTIATE_TINYGEMM_TEMPLATE(TYPE) \
  template void tinygemm_kernel<TYPE>(      \
      const uint8_t* __restrict__ A,        \
      const int8_t* __restrict__ B,         \
      TYPE* __restrict__ C,                 \
      int32_t* __restrict__ Ctmp,           \
      const float* __restrict__ As,         \
      const float* __restrict__ Bs,         \
      int64_t M,                            \
      int64_t N,                            \
      int64_t K,                            \
      int64_t lda,                          \
      int64_t ldb,                          \
      int64_t ldc,                          \
      bool brg)

INSTANTIATE_TINYGEMM_TEMPLATE(at::BFloat16);
INSTANTIATE_TINYGEMM_TEMPLATE(at::Half);

std::tuple<at::Tensor, at::Tensor> per_token_quant_int8_cpu(at::Tensor& A) {
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(A);
  CHECK_DIM(2, A);
```
**EN:** This section uses `per_token_quant_int8_cpu`, `INSTANTIATE_TINYGEMM_TEMPLATE`, `CHECK_LAST_DIM_CONTIGUOUS_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`per_token_quant_int8_cpu`、`INSTANTIATE_TINYGEMM_TEMPLATE`、`CHECK_LAST_DIM_CONTIGUOUS_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 386-409: Runtime integration and dispatch
```cpp
  int64_t M = A.size(0);
  int64_t K = A.size(1);
  int64_t lda = A.stride(0);

  const auto st = A.scalar_type();
  TORCH_CHECK(st == at::kBFloat16 || st == at::kHalf, "per_token_quant_int8: expect A to be bfloat16 or half.");

  auto Aq = at::empty({M, K}, A.options().dtype(at::kByte));
  auto As = at::empty({M}, A.options().dtype(at::kFloat));

  AT_DISPATCH_REDUCED_FLOATING_TYPES(st, "per_token_quant_int8", [&] {
    uint8_t* __restrict__ Aq_data = Aq.data_ptr<uint8_t>();
    float* __restrict__ As_data = As.data_ptr<float>();
    const scalar_t* __restrict__ A_data = A.data_ptr<scalar_t>();

    at::parallel_for(0, M, 0, [&](int64_t begin, int64_t end) {
      for (int64_t m = begin; m < end; ++m) {
        quantize_row_int8<scalar_t>(Aq_data + m * K, As_data[m], A_data + m * lda, K);
      }
    });
  });
  return std::make_tuple(Aq, As);
}
```
**EN:** This section uses `at::parallel_for`, `stride`, `scalar_type` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`stride`、`scalar_type`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 410-436: Runtime integration and dispatch
```cpp
// weight     :  static, per-channel, symmetric
// activation : dynamic,   per-token, symmetric
//
// mat1    : [M, K]
// mat2    : [N, K]
// scales1 : [M]
// scales2 : [N]
// bias    : [N]
// out     : [M, N]
//
at::Tensor int8_scaled_mm_cpu(
    at::Tensor& mat1,
    at::Tensor& mat2,
    at::Tensor& scales1,
    at::Tensor& scales2,
    const std::optional<at::Tensor>& bias,
    at::ScalarType out_dtype,
    bool is_vnni) {
  auto packed_w = is_vnni ? mat2 : convert_weight_packed(mat2);

  CHECK_INPUT(mat1);
  CHECK_INPUT(mat2);
  CHECK_INPUT(scales1);
  CHECK_INPUT(scales2);
  CHECK_DIM(2, mat1);
  CHECK_DIM(2, mat2);
```
**EN:** This section uses `int8_scaled_mm_cpu`, `convert_weight_packed`, `CHECK_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`int8_scaled_mm_cpu`、`convert_weight_packed`、`CHECK_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 437-460: Runtime integration and dispatch
```cpp
  int64_t M = mat1.size(0);
  int64_t N = mat2.size(0);
  int64_t K = mat1.size(1);

  // see [NOTE]: s8s8 igemm compensation in avx512-vnni
  CHECK_EQ(mat2.size(1), (int64_t)(is_vnni ? K + sizeof(int32_t) : K));
  CHECK_EQ(scales1.numel(), M);
  CHECK_EQ(scales2.numel(), N);

  TORCH_CHECK(mat1.scalar_type() == at::kByte, "int8_scaled_mm: expect mat1 to be uint8.");
  TORCH_CHECK(mat2.scalar_type() == at::kChar, "int8_scaled_mm: expect mat2 to be int8.");
  TORCH_CHECK(
      scales1.scalar_type() == at::kFloat && scales2.scalar_type() == at::kFloat,
      "int8_scaled_mm: expect scales to be float32.");

  auto out = at::empty({M, N}, mat1.options().dtype(out_dtype));

  const bool has_bias = bias.has_value();
  const float* bias_data = nullptr;
  if (has_bias) {
    CHECK_EQ(bias.value().size(0), N);
    bias_data = bias.value().data_ptr<float>();
  }
```
**EN:** This section uses `CHECK_EQ`, `TORCH_CHECK`, `options` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_EQ`、`TORCH_CHECK`、`options`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 461-486: Runtime integration and dispatch
```cpp
  AT_DISPATCH_REDUCED_FLOATING_TYPES(out_dtype, "int8_scaled_mm_kernel_impl", [&] {
    int8_scaled_mm_kernel_impl<scalar_t>(
        out.data_ptr<scalar_t>(),
        mat1.data_ptr<uint8_t>(),
        packed_w.data_ptr<int8_t>(),
        scales1.data_ptr<float>(),
        scales2.data_ptr<float>(),
        bias_data,
        M,
        N,
        K);
  });
  return out;
}

#ifndef __aarch64__
// fused `per_token_quant_int8_cpu` and `int8_scaled_mm_cpu`
at::Tensor int8_scaled_mm_with_quant(
    at::Tensor& mat1,
    at::Tensor& mat2,
    at::Tensor& scales2,
    const std::optional<at::Tensor>& bias,
    at::ScalarType out_dtype,
    bool is_vnni) {
  auto packed_w = is_vnni ? mat2 : convert_weight_packed(mat2);
```
**EN:** This section uses `int8_scaled_mm_with_quant`, `convert_weight_packed` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`int8_scaled_mm_with_quant`、`convert_weight_packed`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 487-511: Runtime integration and dispatch
```cpp
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(mat1);
  CHECK_INPUT(mat2);
  CHECK_INPUT(scales2);
  CHECK_DIM(2, mat1);
  CHECK_DIM(2, mat2);

  int64_t M = mat1.size(0);
  int64_t N = mat2.size(0);
  int64_t K = mat1.size(1);
  int64_t lda = mat1.stride(0);

  // see [NOTE]: s8s8 igemm compensation in avx512-vnni
  CHECK_EQ(mat2.size(1), (int64_t)(is_vnni ? K + sizeof(int32_t) : K));
  CHECK_EQ(scales2.numel(), N);

  const auto st = mat1.scalar_type();
  TORCH_CHECK(st == at::kBFloat16 || st == at::kHalf, "int8_scaled_mm_with_quant: expect A to be bfloat16 or half.");
  TORCH_CHECK(st == out_dtype, "int8_scaled_mm_with_quant: expect A has same dtype with out_dtype.");
  TORCH_CHECK(mat2.scalar_type() == at::kChar, "int8_scaled_mm_with_quant: expect mat2 to be int8.");
  TORCH_CHECK(scales2.scalar_type() == at::kFloat, "int8_scaled_mm_with_quant: expect scales to be float32.");

  const int64_t buffer_size = M * K + M * sizeof(float);
  auto buffer = at::empty({buffer_size}, mat1.options().dtype(at::kByte));
  auto out = at::empty({M, N}, mat1.options().dtype(out_dtype));
```
**EN:** This section uses `CHECK_LAST_DIM_CONTIGUOUS_INPUT`, `CHECK_INPUT`, `CHECK_DIM` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_LAST_DIM_CONTIGUOUS_INPUT`、`CHECK_INPUT`、`CHECK_DIM`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 512-529: Runtime integration and dispatch
```cpp
  const bool has_bias = bias.has_value();
  const float* bias_data = nullptr;
  if (has_bias) {
    CHECK_EQ(bias.value().size(0), N);
    bias_data = bias.value().data_ptr<float>();
  }

  AT_DISPATCH_REDUCED_FLOATING_TYPES(out_dtype, "int8_scaled_mm_with_quant_kernel_impl", [&] {
    uint8_t* __restrict__ Aq_data = buffer.data_ptr<uint8_t>();
    float* __restrict__ As_data = (float*)((void*)(Aq_data + M * K));
    const scalar_t* __restrict__ A_data = mat1.data_ptr<scalar_t>();

    at::parallel_for(0, M, 0, [&](int64_t begin, int64_t end) {
      for (int64_t m = begin; m < end; ++m) {
        quantize_row_int8<scalar_t>(Aq_data + m * K, As_data[m], A_data + m * lda, K);
      }
    });
```
**EN:** This section uses `at::parallel_for`, `has_value`, `CHECK_EQ` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`has_value`、`CHECK_EQ`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 530-542: Local implementation details
```cpp
    int8_scaled_mm_kernel_impl<scalar_t>(
        out.data_ptr<scalar_t>(),
        Aq_data,
        packed_w.data_ptr<int8_t>(),
        As_data,
        scales2.data_ptr<float>(),
        bias_data,
        M,
        N,
        K);
  });
  return out;
}
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 543-543: Local implementation details
```cpp
#endif  // #ifndef __aarch64__
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `common.h`, `gemm.h`, `vec.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cpu / gemm_int8.cpp
