# cpu_fused_moe.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/cpu_fused_moe.cpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements fused CPU Mixture-of-Experts routing, activation, and expert GEMM orchestration. / 实现融合式 CPU 混合专家路由、激活和专家 GEMM 调度。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-8)
```cpp
#include "cpu/cpu_types.hpp"
#include "cpu/utils.hpp"
#include "cpu/micro_gemm/cpu_micro_gemm_vec.hpp"
#include "cpu/cpu_arch_macros.h"

#ifdef CPU_CAPABILITY_AMXBF16
  #include "cpu/micro_gemm/cpu_micro_gemm_amx.hpp"
  #define AMX_DISPATCH(...)                                                    \
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: gelu_and_mul (lines 109-144)
```cpp
template <typename scalar_t>
void gelu_and_mul(float* __restrict__ input, scalar_t* __restrict__ output,
                  const int32_t m_size, const int32_t n_size,
                  const int32_t input_stride, const int32_t output_stride) {
  using scalar_vec_t = typename cpu_utils::VecTypeTrait<scalar_t>::vec_t;
  const int32_t dim = n_size / 2;
  float* __restrict__ gate = input;
  float* __restrict__ up = input + dim;
  vec_op::FP32Vec16 one_vec(1.0);
  vec_op::FP32Vec16 w1_vec(M_SQRT1_2);
  vec_op::FP32Vec16 w2_vec(0.5);
  alignas(64) float temp[16];

  DEFINE_FAST_EXP
// ...
    output += output_stride;
  }
}
```
**EN:** This block computes a fused activation path so activation and gating can be applied with minimal extra memory traffic.
**CN:** 该代码块实现融合激活路径，使激活与门控能够在尽量少的额外内存访问下完成。

### Function / Kernel: prepack_moe_weight_impl (lines 168-181)
```cpp
template <typename scalar_t, typename gemm_t>
void prepack_moe_weight_impl(scalar_t* __restrict__ weight_ptr,
                             scalar_t* __restrict__ packed_weight_ptr,
                             const int32_t expert_num,
                             const int32_t output_size,
                             const int32_t input_size,
                             const int64_t expert_stride) {
#pragma omp parallel for
  for (int32_t e_idx = 0; e_idx < expert_num; ++e_idx) {
    gemm_t::pack_weight(weight_ptr + expert_stride * e_idx,
                        packed_weight_ptr + expert_stride * e_idx, output_size,
                        input_size);
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: fused_moe_impl (lines 183-711)
```cpp
template <typename scalar_t, typename w_t, typename gemm_t>
void fused_moe_impl(scalar_t* __restrict__ output, scalar_t* __restrict__ input,
                    w_t* __restrict__ w13, w_t* __restrict__ w2,
                    w_t* __restrict__ w13_bias, w_t* __restrict__ w2_bias,
                    float* __restrict__ topk_weights,
                    int32_t* __restrict__ topk_id, FusedMOEAct act_type,
                    const int32_t token_num, const int32_t expert_num,
                    const int32_t topk_num, const int32_t input_size_13,
                    const int32_t output_size_13, const int32_t input_size_2,
                    const int32_t output_size_2, const bool skip_weighted) {
  using scalar_vec_t = typename cpu_utils::VecTypeTrait<scalar_t>::vec_t;
  constexpr int32_t gemm_n_tile_size = gemm_t::NSize;
  constexpr int32_t gemm_m_tile_size = gemm_t::MaxMSize;
  constexpr int32_t min_w13_n_tile_size = 2 * gemm_n_tile_size;
// ...
    }
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: prepack_moe_weight (lines 714-735)
```cpp
void prepack_moe_weight(
    const torch::Tensor& weight,  // [expert_num, output_size, input_size]
    torch::Tensor& packed_weight, const std::string& isa) {
  TORCH_CHECK(weight.is_contiguous());
  const int32_t expert_num = weight.size(0);
  const int32_t output_size = weight.size(1);
  const int32_t input_size = weight.size(2);
  TORCH_CHECK_EQ(output_size % 32, 0);
  const int64_t expert_stride = weight.stride(0);
  cpu_utils::ISA isa_type = cpu_utils::get_isa(isa);

  VLLM_DISPATCH_FLOATING_TYPES(
      weight.scalar_type(), "prepack_moe_weight", [&]() {
        CPU_ISA_DISPATCH_IMPL(isa_type, [&]() {
// ...
        });
      });
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: cpu_fused_moe (lines 737-776)
```cpp
void cpu_fused_moe(
    torch::Tensor& output,       // [token_num, output_size_2]
    const torch::Tensor& input,  // [token_num, input_size_13]
    const torch::Tensor&
        w13,  // [expert_num, output_size_13, input_size_13], packed
    const torch::Tensor&
        w2,  // [expert_num, output_size_2, input_size_2], packed
    const std::optional<torch::Tensor>&
        w13_bias,  // [expert_num, output_size_13]
    const std::optional<torch::Tensor>& w2_bias,  // [expert_num, output_size_2]
    const torch::Tensor& topk_weights,            // [token_num, k], float32
    const torch::Tensor& topk_id,                 // [token_num, k], int32
    const bool skip_weighted, const std::string& act, const std::string& isa) {
  const int32_t token_num = input.size(0);
// ...
    });
  });
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Top-k selection / Top-k 选择
- Tiled matrix multiplication / 分块矩阵乘法
- Integer kernel specialization / 整数内核专用化
- Intel AMX tiling / Intel AMX 分块
- SIMD vectorization / SIMD 向量化
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cpu/cpu_types.hpp`, `cpu/utils.hpp`, `cpu/micro_gemm/cpu_micro_gemm_vec.hpp`, `cpu/cpu_arch_macros.h`, `cpu/micro_gemm/cpu_micro_gemm_amx.hpp`
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
