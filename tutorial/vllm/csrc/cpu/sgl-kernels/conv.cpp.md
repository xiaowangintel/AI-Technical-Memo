# conv.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/sgl-kernels/conv.cpp`
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
#include "vec.h"
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Struct: tinygemm_kernel (lines 71-207)
```cpp
struct tinygemm_kernel<at::BFloat16, K, BLOCK_N, has_bias, has_silu> {
  static inline void apply(
      const at::BFloat16* __restrict__ A,
      const at::BFloat16* __restrict__ B,
      at::BFloat16* __restrict__ C,
      const at::BFloat16* __restrict__ bias,
      const at::BFloat16* __restrict__ conv_states,
      bool has_initial_state,
      int64_t M,
      int64_t lda,
      bool is_first_token) {
    assert(K == 4);
    constexpr int ROWS = K;
    constexpr int COLS = BLOCK_N / block_size_n();
// ...
    }
  }
};
```
**EN:** This type defines shared state, data layout, or a reusable helper abstraction for the rest of the file.
**CN:** 该类型定义了共享状态、数据布局或供本文件其他部分复用的辅助抽象。

### Function / Kernel: causal_conv1d_fwd_kernel_impl (lines 222-290)
```cpp
template <typename scalar_t>
void causal_conv1d_fwd_kernel_impl(
    scalar_t* __restrict__ out,
    const scalar_t* __restrict__ input,
    const scalar_t* __restrict__ weight,
    const scalar_t* __restrict__ bias,
    scalar_t* __restrict__ conv_states,
    const int32_t* __restrict__ conv_indices,
    const bool* __restrict__ has_initial_state,
    bool silu_activation,
    int64_t batch,
    int64_t dim,
    int64_t seqlen,
    int64_t width,
// ...
    });
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: causal_conv1d_fwd_varlen_kernel_impl (lines 305-380)
```cpp
template <typename scalar_t>
void causal_conv1d_fwd_varlen_kernel_impl(
    scalar_t* __restrict__ out,
    const scalar_t* __restrict__ input,
    const scalar_t* __restrict__ weight,
    const scalar_t* __restrict__ bias,
    scalar_t* __restrict__ conv_states,
    const int32_t* __restrict__ query_start_loc,
    const int32_t* __restrict__ conv_indices,
    const bool* __restrict__ has_initial_state,
    const int32_t* __restrict__ block_indices,
    bool silu_activation,
    int64_t batch,
    int64_t dim,
// ...
    });
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: causal_conv1d_update_kernel_impl (lines 382-450)
```cpp
template <typename scalar_t>
void causal_conv1d_update_kernel_impl(
    scalar_t* __restrict__ out,
    const scalar_t* __restrict__ input,
    scalar_t* __restrict__ conv_states,
    const scalar_t* __restrict__ weight,
    const scalar_t* __restrict__ bias,
    const int32_t* __restrict__ conv_indices,
    bool silu_activation,
    int64_t batch,
    int64_t dim,
    int64_t seqlen,
    int64_t width,
    int64_t conv_state_slot_stride) {
// ...
    }
  });
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: causal_conv1d_weight_pack (lines 456-488)
```cpp
at::Tensor causal_conv1d_weight_pack(const at::Tensor& weight) {
  CHECK_INPUT(weight);

  int64_t dim = weight.size(0);
  int64_t width = weight.size(1);
  constexpr int64_t BLOCK_N = block_size_n();
  TORCH_CHECK(width == 4, "causal_conv1d_weight_pack: support only width of 4");
  TORCH_CHECK(dim % BLOCK_N == 0, "causal_conv1d_weight_pack: invalid dim size ", dim);

  const int64_t N = dim, K2 = width >> 1;
  const int64_t NB = div_up(N, BLOCK_N);

  auto packed_weight = at::empty_like(weight);
  AT_DISPATCH_REDUCED_FLOATING_TYPES(weight.scalar_type(), "causal_conv1d_fwd_kernel_impl", [&] {
// ...
  });
  return packed_weight;
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

## Key Concepts / 关键概念
- Tiled matrix multiplication / 分块矩阵乘法
- Fused activation functions / 融合激活函数
- SIMD vectorization / SIMD 向量化
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
