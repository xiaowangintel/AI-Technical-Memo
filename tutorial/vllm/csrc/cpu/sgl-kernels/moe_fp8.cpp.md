# moe_fp8.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/sgl-kernels/moe_fp8.cpp`
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

### Function / Kernel: fused_experts_fp_kernel_impl (lines 10-214)
```cpp
template <typename scalar_t, typename packed_t, typename param_t, bool is_mxfp4>
void fused_experts_fp_kernel_impl(
    scalar_t* __restrict__ output,
    scalar_t* __restrict__ ic0,
    scalar_t* __restrict__ ic1,
    scalar_t* __restrict__ ic2,
    scalar_t* __restrict__ A_tmp,
    scalar_t* __restrict__ B_tmp,
    float* __restrict__ C_tmp,
    const scalar_t* __restrict__ input,
    const packed_t* __restrict__ packed_w1,
    const packed_t* __restrict__ packed_w2,
    const float* __restrict__ w1_bias,
    const float* __restrict__ w2_bias,
// ...
    }
  });
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Compile-time setup: INSTANTIATE_MOE_FP_TEMPLATE (lines 216-247)
```cpp
#define INSTANTIATE_MOE_FP_TEMPLATE(TYPE1, TYPE2, TYPE3, IS_MXFP4)           \
  template void fused_experts_fp_kernel_impl<TYPE1, TYPE2, TYPE3, IS_MXFP4>( \
      TYPE1* __restrict__ output,                                            \
      TYPE1* __restrict__ ic0,                                               \
      TYPE1* __restrict__ ic1,                                               \
      TYPE1* __restrict__ ic2,                                               \
      TYPE1* __restrict__ A_tmp,                                             \
      TYPE1* __restrict__ B_tmp,                                             \
      float* __restrict__ C_tmp,                                             \
      const TYPE1* __restrict__ input,                                       \
      const TYPE2* __restrict__ packed_w1,                                   \
      const TYPE2* __restrict__ packed_w2,                                   \
      const float* __restrict__ w1_bias,                                     \
      const float* __restrict__ w2_bias,                                     \
// ...
      float limit,                                                           \
      CPUAcTMethod act_func,                                                 \
      bool with_bias)
```
**EN:** This block defines compile-time constants or helper macros that shape tiling, vector width, or backend-specific behavior.
**CN:** 该代码块定义了编译期常量或辅助宏，用来决定分块方式、向量宽度或后端特定行为。

### Function / Kernel: shared_expert_fp8_kernel_impl (lines 254-381)
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
// ...
    at::native::cpublas::brgemm_release();
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Compile-time setup: INSTANTIATE_SHARED_EXPERT_FP8_TEMPLATE (lines 383-401)
```cpp
#define INSTANTIATE_SHARED_EXPERT_FP8_TEMPLATE(TYPE)   \
  template void shared_expert_fp8_kernel_impl<TYPE>(   \
      TYPE* __restrict__ output,                       \
      TYPE* __restrict__ ic0,                          \
      TYPE* __restrict__ ic1,                          \
      TYPE* __restrict__ B_tmp,                        \
      float* __restrict__ C_tmp,                       \
      const TYPE* __restrict__ input,                  \
      const at::Float8_e4m3fn* __restrict__ packed_w1, \
      const at::Float8_e4m3fn* __restrict__ packed_w2, \
      const float* __restrict__ w1s,                   \
      const float* __restrict__ w2s,                   \
      int64_t block_size_N,                            \
      int64_t block_size_K,                            \
// ...
      int64_t M,                                       \
      int64_t N,                                       \
      int64_t K)
```
**EN:** This block defines compile-time constants or helper macros that shape tiling, vector width, or backend-specific behavior.
**CN:** 该代码块定义了编译期常量或辅助宏，用来决定分块方式、向量宽度或后端特定行为。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Top-k selection / Top-k 选择
- Tiled matrix multiplication / 分块矩阵乘法
- FP8 quantization / compute / FP8 量化与计算
- Integer kernel specialization / 整数内核专用化
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
