# gemm_int4.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/sgl-kernels/gemm_int4.cpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements SGLang-style CPU kernels for GEMM, MoE, vector ops, and related utilities. / 实现 SGLang 风格的 CPU 内核，涵盖 GEMM、MoE、向量运算及相关工具。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-9)
```cpp
// Adapted from
// https://github.com/sgl-project/sglang/tree/main/sgl-kernel/csrc/cpu

// clang-format off

#include <torch/all.h>

#include "gemm.h"
#include "vec.h"
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: _dequant_and_store (lines 87-141)
```cpp
template <bool accum, int64_t N, bool sym_quant_act>
void _dequant_and_store(
    float* __restrict__ output,
    const int32_t* __restrict__ input,
    const float* __restrict__ scale_a,
    const int32_t* __restrict__ zp_a,
    const float* __restrict__ scale_b,
    const int32_t* __restrict__ comp_b,
    int M,
    int ldi,
    int ldo,
    int ldsa = 1) {
  for (int m = 0; m < M; ++m) {
    float a_scale = *(scale_a + m * ldsa);
// ...
    }
  }
}
```
**EN:** This function quantizes or dequantizes expert data, typically preparing compact low-precision buffers for faster compute.
**CN:** 该函数对专家数据进行量化或反量化，通常用于准备更紧凑的低精度缓冲区以提升计算效率。

### Function / Kernel: _dequant_gemm_accum_small_M (lines 175-265)
```cpp
template <int64_t M, int64_t N, int64_t ldb, bool sym_quant_act>
void _dequant_gemm_accum_small_M(
    float* __restrict__ C,
    const uint8_t* A,
    const float* scales_a,
    const int32_t* qzeros_a,
    const uint8_t* B,
    const float* scales_b,
    const int8_t* qzeros_b,
    int64_t K,
    int64_t lda,
    int64_t ldc) {
  // if sym_quant_act is true, A pointer type is passed in as uint8_t* but actually int8_t*.

// ...
  };
  Unroll<M * COLS>{}(store);
}
```
**EN:** This function quantizes or dequantizes expert data, typically preparing compact low-precision buffers for faster compute.
**CN:** 该函数对专家数据进行量化或反量化，通常用于准备更紧凑的低精度缓冲区以提升计算效率。

### Function / Kernel: _dequant_gemm_accum (lines 271-326)
```cpp
template <int64_t N, int64_t ldb, bool sym_quant_act>
void _dequant_gemm_accum(
    float* C,
    const uint8_t* A,
    const float* scales_a,
    const int32_t* qzeros_a,
    const uint8_t* B,
    const float* scales_b,
    const int8_t* qzeros_b,
    const int32_t* compensation,
    int8_t* dqB,
    int64_t M,
    int64_t K,
    int64_t lda,
// ...
    TORCH_CHECK(false, "tinygemm_kernel: scalar path not implemented!");
  }
}
```
**EN:** This function quantizes or dequantizes expert data, typically preparing compact low-precision buffers for faster compute.
**CN:** 该函数对专家数据进行量化或反量化，通常用于准备更紧凑的低精度缓冲区以提升计算效率。

### Function / Kernel: convert_int4_weight_packed_with_compensation (lines 507-596)
```cpp
/*
return: packed_weight, packed_scales, packed_qzeros
*/
std::tuple<at::Tensor, at::Tensor, at::Tensor> convert_int4_weight_packed_with_compensation(
    const at::Tensor& weight, const at::Tensor& scales, const at::Tensor& qzeros) {
  // weight shape = [N, K]
  // scales shape = [N, G]
  // qzeros shape = [N, G]
  TORCH_CHECK(weight.dim() == 2, "DA8W4 CPU: Weight should be a 2D tensor for packing");
  TORCH_CHECK(weight.size(1) % 2 == 0, "DA8W4 CPU: Weight should have even number of columns for packing");

  auto new_scales = scales;
  auto new_qzeros = qzeros;
  if (new_scales.dim() == 1) {
// ...

  return std::make_tuple(std::move(blocked_weight), std::move(blocked_scales), std::move(blocked_qzeros));
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: unpack_4bit_to_32bit_signed (lines 598-639)
```cpp
std::tuple<at::Tensor, at::Tensor> unpack_4bit_to_32bit_signed(const at::Tensor& qweight, const at::Tensor& qzeros) {
  TORCH_CHECK(qweight.scalar_type() == at::kInt, "qweight must be int32");
  TORCH_CHECK(qzeros.scalar_type() == at::kInt, "qzeros must be int32");
  const auto W0 = qweight.size(0);
  const auto W1 = qweight.size(1);
  const auto Z0 = qzeros.size(0);
  const auto Z1 = qzeros.size(1);

  // unpacked_weights: (W0 * 8, W1), int8
  auto unpacked_weights = at::zeros({W0 * 8, W1}, at::TensorOptions().dtype(at::kChar));
  // unpacked_zeros: (Z0, Z1 * 8), int8
  auto unpacked_zeros = at::zeros({Z0, Z1 * 8}, at::TensorOptions().dtype(at::kChar));

  const int32_t* qw_ptr = qweight.data_ptr<int32_t>();
// ...

  return std::make_tuple(unpacked_weights, unpacked_zeros + 1);
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Token permutation / Token 置换
- Tiled matrix multiplication / 分块矩阵乘法
- Low-bit quantization / 低比特量化
- Integer kernel specialization / 整数内核专用化
- Fused activation functions / 融合激活函数
- SIMD vectorization / SIMD 向量化
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: PyTorch / ATen, C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
