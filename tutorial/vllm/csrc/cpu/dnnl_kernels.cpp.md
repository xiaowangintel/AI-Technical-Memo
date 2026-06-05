# dnnl_kernels.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/dnnl_kernels.cpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements CPU kernels backed by oneDNN primitives. / 实现基于 oneDNN 原语的 CPU 内核。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-2)
```cpp
#include "cpu_types.hpp"
#include "dnnl_helper.h"
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: static_scaled_int8_quant_impl (lines 35-89)
```cpp
template <bool AZP, typename scalar_t>
void static_scaled_int8_quant_impl(const scalar_t* input, int8_t* output,
                                   const float* scale, const int32_t* azp,
                                   const int64_t num_tokens,
                                   const int64_t input_stride,
                                   const int64_t hidden_size) {
  using load_vec_t = typename KernelVecType<scalar_t>::load_vec_type;
  using cvt_vec_t = typename KernelVecType<scalar_t>::cvt_vec_type;
  constexpr int64_t vec_elem_num = load_vec_t::VEC_ELEM_NUM;

  constexpr float i8_min =
      static_cast<float>(std::numeric_limits<int8_t>::min());
  constexpr float i8_max =
      static_cast<float>(std::numeric_limits<int8_t>::max());
// ...
    elems_int8.save(output_ptr + j, hidden_size - j);
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: dynamic_scaled_int8_quant_impl (lines 91-192)
```cpp
template <bool AZP, typename scalar_t>
void dynamic_scaled_int8_quant_impl(const scalar_t* input, int8_t* output,
                                    float* scale, int32_t* azp,
                                    const int64_t num_tokens,
                                    const int64_t input_stride,
                                    const int64_t hidden_size) {
  using load_vec_t = typename KernelVecType<scalar_t>::load_vec_type;
  using cvt_vec_t = typename KernelVecType<scalar_t>::cvt_vec_type;
  constexpr int vec_elem_num = load_vec_t::VEC_ELEM_NUM;

  constexpr float i8_min =
      static_cast<float>(std::numeric_limits<int8_t>::min());
  constexpr float i8_max =
      static_cast<float>(std::numeric_limits<int8_t>::max());
// ...
    }
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: dynamic_quant_epilogue (lines 194-300)
```cpp
template <bool AZP, bool Bias, typename scalar_t>
void dynamic_quant_epilogue(const float* input, scalar_t* output,
                            const float* a_scale, const int32_t* azp,
                            const float* azp_adj, const scalar_t* bias,
                            const int64_t num_tokens,
                            const int64_t hidden_size) {
  CPU_KERNEL_GUARD_IN(dynamic_quant_epilogue)
  using load_vec_t = typename KernelVecType<scalar_t>::load_vec_type;
  using cvt_vec_t = typename KernelVecType<scalar_t>::cvt_vec_type;
  constexpr int vec_elem_num = load_vec_t::VEC_ELEM_NUM;

  const int64_t thread_num = omp_get_max_threads();
  if (num_tokens > thread_num) {
#pragma omp parallel for
// ...
    }
  }
}
```
**EN:** This function quantizes or dequantizes expert data, typically preparing compact low-precision buffers for faster compute.
**CN:** 该函数对专家数据进行量化或反量化，通常用于准备更紧凑的低精度缓冲区以提升计算效率。

### Function / Kernel: create_onednn_scaled_mm_handler (lines 303-352)
```cpp
int64_t create_onednn_scaled_mm_handler(
    const torch::Tensor& b,         // [IC, OC], column-major
    const torch::Tensor& b_scales,  // [1] or [OC]
    at::ScalarType output_type, bool dynamic_act_quant, bool use_azp,
    int64_t primitive_cache_size) {
  TORCH_CHECK(b.dim() == 2);
  TORCH_CHECK(b.stride(0) == 1);  // Column-major
  TORCH_CHECK(b_scales.is_contiguous());

  W8A8MatMulPrimitiveHandler::Args args;
  args.primitive_cache_size = primitive_cache_size;

  if (b_scales.numel() == 1) {
    args.b_quantization_strategy =
// ...

  return reinterpret_cast<int64_t>(new W8A8MatMulPrimitiveHandler(args));
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: static_scaled_int8_quant (lines 437-465)
```cpp
void static_scaled_int8_quant(
    torch::Tensor& out,          // [batch, hidden_size]
    const torch::Tensor& input,  // [batch, hidden_size]
    const torch::Tensor& scale, std::optional<torch::Tensor> const& azp) {
  CPU_KERNEL_GUARD_IN(static_scaled_int8_quant)
  TORCH_CHECK(out.is_contiguous());
  TORCH_CHECK_EQ(input.dim(), 2);
  TORCH_CHECK_EQ(input.stride(1), 1);
  TORCH_CHECK(scale.numel() == 1);
  TORCH_CHECK(!azp.has_value() || azp->numel() == 1);

  const int64_t stride = input.stride(0);
  const int64_t hidden_size = input.size(1);
  const int64_t num_tokens = input.size(0);
// ...
        }
      });
}
```
**EN:** This function quantizes or dequantizes expert data, typically preparing compact low-precision buffers for faster compute.
**CN:** 该函数对专家数据进行量化或反量化，通常用于准备更紧凑的低精度缓冲区以提升计算效率。

### Function / Kernel: dynamic_scaled_int8_quant (lines 468-495)
```cpp
void dynamic_scaled_int8_quant(
    torch::Tensor& out,          // [batch, hidden_size]
    const torch::Tensor& input,  // [batch, hidden_size]
    torch::Tensor& scale,        // [batch, 1]
    std::optional<torch::Tensor> const& azp) {
  CPU_KERNEL_GUARD_IN(dynamic_scaled_int8_quant)
  TORCH_CHECK(out.is_contiguous());
  TORCH_CHECK_EQ(input.dim(), 2);
  TORCH_CHECK_EQ(input.stride(1), 1);

  const int64_t hidden_size = input.size(1);
  const int64_t num_tokens = input.size(0);
  const int64_t stride = input.stride(0);
  VLLM_DISPATCH_FLOATING_TYPES(
// ...
        }
      });
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

## Key Concepts / 关键概念
- Integer kernel specialization / 整数内核专用化
- oneDNN integration / oneDNN 集成
- SIMD vectorization / SIMD 向量化
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: C++ standard library or platform support, oneDNN
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
