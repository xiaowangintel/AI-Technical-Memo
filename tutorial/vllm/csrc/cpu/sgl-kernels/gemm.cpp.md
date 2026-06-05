# gemm.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/sgl-kernels/gemm.cpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements SGLang-style CPU kernels for GEMM, MoE, vector ops, and related utilities. / 实现 SGLang 风格的 CPU 内核，涵盖 GEMM、MoE、向量运算及相关工具。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-9)
```cpp
// Adapted from
// https://github.com/sgl-project/sglang/tree/main/sgl-kernel/csrc/cpu

// clang-format off

#include "gemm.h"

#include "common.h"
#include "vec.h"
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: weight_packed_linear_kernel_impl (lines 462-512)
```cpp
template <typename scalar_t>
void weight_packed_linear_kernel_impl(
    scalar_t* __restrict__ out,
    const scalar_t* __restrict__ mat1,
    const scalar_t* __restrict__ mat2,
    const float* __restrict__ bias,
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t mat1_strideM,
    int64_t out_strideM) {
  constexpr int64_t BLOCK_M = block_size_m();
  constexpr int64_t BLOCK_N = block_size_n();
  const int64_t MB = div_up(M, BLOCK_M);
// ...
    });
  });
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: weight_packed_linear_kernel_impl (lines 514-587)
```cpp
template <typename scalar_t>
void weight_packed_linear_kernel_impl(
    scalar_t* __restrict__ out,
    const scalar_t* __restrict__ mat1,
    const float* __restrict__ mat2,
    const float* __restrict__ bias,
    const scalar_t* __restrict__ post_mul_mat,
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t mat1_strideM,
    int64_t out_strideM) {
  constexpr int64_t BLOCK_M = block_size_m();
  constexpr int64_t BLOCK_N = block_size_n();
// ...
    });
  });
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: convert_weight_packed (lines 625-693)
```cpp
at::Tensor convert_weight_packed(at::Tensor& weight) {
  // for 3d moe weights
  // weight : [E, OC, IC]
  //     w1 : [E, 2N,  K]
  //     w2 : [E,  K,  N]
  CHECK_INPUT(weight);

  const int64_t ndim = weight.ndimension();
  TORCH_CHECK(ndim == 2 || ndim == 3, "expect weight to be 2d or 3d, got ", ndim, "d tensor.");

  if (ndim == 2 && weight.size(0) < TILE_N) {
    // for 2D weight and small OC shape, we use fma linear path, which needs transpose not pack
    return weight.to(at::kFloat).t().contiguous();
  }
// ...
  });
  return packed_weight;
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: convert_scale_packed (lines 695-728)
```cpp
at::Tensor convert_scale_packed(at::Tensor& scale) {
  CHECK_INPUT(scale);

  const int64_t ndim = scale.ndimension();
  TORCH_CHECK(ndim == 2 || ndim == 3, "expect scale to be 2d or 3d, got ", ndim, "d tensor.");
  const auto st = scale.scalar_type();
  const int64_t E = ndim == 3 ? scale.size(0) : 1;
  const int64_t N = ndim == 3 ? scale.size(1) : scale.size(0);
  // number of groups, e.g. K/32
  const int64_t G = ndim == 3 ? scale.size(2) : scale.size(1);

  constexpr int64_t BLOCK_N = block_size_n();
  TORCH_CHECK(N % BLOCK_N == 0, "invalid weight out features ", N);
  const int64_t NB = N / BLOCK_N;
// ...
  });
  return packed_scale;
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: weight_packed_linear (lines 735-796)
```cpp
at::Tensor
weight_packed_linear(at::Tensor& mat1, at::Tensor& mat2, const std::optional<at::Tensor>& bias, bool is_vnni) {
  auto packed_w = is_vnni ? mat2 : convert_weight_packed(mat2);
  bool use_fma_gemm = false;
  if (packed_w.scalar_type() == at::kFloat) {
    use_fma_gemm = true;
  }

  int64_t M = mat1.size(0);
  int64_t K = mat1.size(1);
  int64_t N = use_fma_gemm ? mat2.size(1) : mat2.size(0);

  CHECK_LAST_DIM_CONTIGUOUS_INPUT(mat1);
  CHECK_INPUT(mat2);
// ...

  return out;
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Tiled matrix multiplication / 分块矩阵乘法
- FP8 quantization / compute / FP8 量化与计算
- Low-bit quantization / 低比特量化
- Integer kernel specialization / 整数内核专用化
- SIMD vectorization / SIMD 向量化
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
