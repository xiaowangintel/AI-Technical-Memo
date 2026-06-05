# moe_int4.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/sgl-kernels/moe_int4.cpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements SGLang-style CPU kernels for GEMM, MoE, vector ops, and related utilities. / 实现 SGLang 风格的 CPU 内核，涵盖 GEMM、MoE、向量运算及相关工具。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-8)
```cpp
// Adapted from
// https://github.com/sgl-project/sglang/tree/main/sgl-kernel/csrc/cpu

// clang-format off

#include "common.h"
#include "gemm.h"
#include "moe.h"
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: copy_bias (lines 10-24)
```cpp
template <int64_t N>
inline void copy_bias(const float* bias_ptr, float* y_buf, int64_t m, int64_t ldn) {
  using Vec = at::vec::Vectorized<float>;
  constexpr int kVecSize = Vec::size();
  static_assert(N % kVecSize == 0, "copy_bias requires N to be a multiple of Vectorized<float>::size()");
  const bool has_bias = bias_ptr != nullptr;
  const Vec zero_vec(0.f);
  for (int i = 0; i < m; ++i) {
#pragma GCC unroll 2
    for (int j = 0; j < N; j += kVecSize) {
      Vec vec = has_bias ? Vec::loadu(bias_ptr + j) : zero_vec;
      vec.store(y_buf + i * ldn + j);
    }
  }
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: fused_experts_int4_w4a8_kernel_impl (lines 26-289)
```cpp
template <typename scalar_t>
void fused_experts_int4_w4a8_kernel_impl(
    scalar_t* __restrict__ output,
    scalar_t* __restrict__ ic0,
    scalar_t* __restrict__ ic1,
    scalar_t* __restrict__ ic2,
    uint8_t* __restrict__ A_tmp,
    uint8_t* __restrict__ Aq_tmp,
    float* __restrict__ As_tmp,
    int32_t* __restrict__ Azp_tmp,
    float* __restrict__ C_tmp,
    int8_t* __restrict__ dqB_tmp,
    const scalar_t* __restrict__ input,
    const uint8_t* __restrict__ packed_w1,
// ...
    }
  });
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Compile-time setup: INSTANTIATE_MOE_INT4_W4A8_TEMPLATE (lines 291-320)
```cpp
#define INSTANTIATE_MOE_INT4_W4A8_TEMPLATE(TYPE)           \
  template void fused_experts_int4_w4a8_kernel_impl<TYPE>( \
      TYPE* __restrict__ output,                           \
      TYPE* __restrict__ ic0,                              \
      TYPE* __restrict__ ic1,                              \
      TYPE* __restrict__ ic2,                              \
      uint8_t* __restrict__ A_tmp,                         \
      uint8_t* __restrict__ Aq_tmp,                        \
      float* __restrict__ As_tmp,                          \
      int32_t* __restrict__ Azp_tmp,                       \
      float* __restrict__ C_tmp,                           \
      int8_t* __restrict__ dqB_tmp,                        \
      const TYPE* __restrict__ input,                      \
      const uint8_t* __restrict__ packed_w1,               \
// ...
      int64_t E,                                           \
      int64_t topk,                                        \
      int64_t num_tokens_post_pad)
```
**EN:** This block defines compile-time constants or helper macros that shape tiling, vector width, or backend-specific behavior.
**CN:** 该代码块定义了编译期常量或辅助宏，用来决定分块方式、向量宽度或后端特定行为。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Top-k selection / Top-k 选择
- Tiled matrix multiplication / 分块矩阵乘法
- Low-bit quantization / 低比特量化
- Integer kernel specialization / 整数内核专用化
- SIMD vectorization / SIMD 向量化
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
