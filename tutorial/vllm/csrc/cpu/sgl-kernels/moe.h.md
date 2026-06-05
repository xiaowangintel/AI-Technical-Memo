# moe.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/sgl-kernels/moe.h`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements SGLang-style CPU kernels for GEMM, MoE, vector ops, and related utilities. / 实现 SGLang 风格的 CPU 内核，涵盖 GEMM、MoE、向量运算及相关工具。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-7)
```cpp
// Adapted from
// https://github.com/sgl-project/sglang/tree/main/sgl-kernel/csrc/cpu

// clang-format off

#pragma once
#include "vec.h"
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: copy_mul_stub (lines 54-75)
```cpp
template <typename scalar_t, typename input_t>
inline void copy_mul_stub(scalar_t* __restrict__ out, const input_t* __restrict__ input, float weight, int64_t size) {
  static_assert(
      std::is_same_v<input_t, float> || std::is_same_v<input_t, scalar_t>,
      "copy_mul_stub only supports input_t == float or input_t == scalar_t");
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;
  constexpr int kVecSize = bVec::size();
  const fVec weight_vec = fVec(weight);
  int64_t d;
#pragma GCC unroll 4
  for (d = 0; d <= size - kVecSize; d += kVecSize) {
    auto [x0, x1] = load_float_vec2(input + d);
    x0 = x0 * weight_vec;
// ...
    out[d] = static_cast<scalar_t>(input[d] * weight);
  }
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: sum_stub (lines 78-112)
```cpp
template <typename scalar_t>
inline void sum_stub(scalar_t* __restrict__ out, const scalar_t* __restrict__ input, int64_t topk, int64_t K) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;
  constexpr int kVecSize = bVec::size();
  if (topk == 1) {
    // do copy for topk = 1
    copy_stub(out, input, K);
  } else {
    // do sum for topk != 1
    int64_t d;
#pragma GCC unroll 4
    for (d = 0; d <= K - kVecSize; d += kVecSize) {
      fVec sum_fvec0 = fVec(0.f);
// ...
    }
  }
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

### Function / Kernel: add_mul_stub (lines 115-153)
```cpp
template <typename scalar_t, typename input_t>
inline void add_mul_stub(
    scalar_t* __restrict__ out,
    const input_t* __restrict__ input,
    const scalar_t* __restrict__ input2,
    float scale,
    int64_t size) {
  static_assert(
      std::is_same_v<input_t, float> || std::is_same_v<input_t, scalar_t>,
      "add_mul_stub only supports input_t == float or input_t == scalar_t");

  // out = input (without scale factor)
  if (input2 == nullptr) {
    copy_stub(out, input, size);
// ...
    out[d] = static_cast<scalar_t>(input[d] + float(input2[d]) * scale);
  }
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: silu_and_mul_stub (lines 155-178)
```cpp
template <typename scalar_t>
inline void silu_and_mul_stub(
    scalar_t* __restrict__ out, const scalar_t* __restrict__ input, const scalar_t* __restrict__ input2, int64_t size) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;
  const fVec one = fVec(1.f);

  // no remainder
#pragma GCC unroll 4
  for (int64_t d = 0; d < size; d += bVec::size()) {
    bVec x = bVec::loadu(input + d);
    fVec x0, x1;
    std::tie(x0, x1) = at::vec::convert_to_float(x);
    bVec y = bVec::loadu(input2 + d);
// ...
    out_vec.store(out + d);
  }
}
```
**EN:** This block computes a fused activation path so activation and gating can be applied with minimal extra memory traffic.
**CN:** 该代码块实现融合激活路径，使激活与门控能够在尽量少的额外内存访问下完成。

### Function / Kernel: copy_mul_stub (lines 216-236)
```cpp
template <typename scalar_t>
inline void copy_mul_stub(scalar_t* __restrict__ out, const scalar_t* __restrict__ input, float weight, int64_t size) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;
  constexpr int kVecSize = bVec::size();
  const fVec weight_vec = fVec(weight);
  int64_t d;
#pragma GCC unroll 4
  for (d = 0; d <= size - kVecSize; d += kVecSize) {
    bVec x = bVec::loadu(input + d);
    fVec x0, x1;
    std::tie(x0, x1) = at::vec::convert_to_float(x);
    x0 = x0 * weight_vec;
    x1 = x1 * weight_vec;
// ...
    out[d] = static_cast<scalar_t>(input[d] * weight);
  }
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: clamp_sigmoid_and_mul_stub (lines 238-284)
```cpp
template <typename scalar_t>
inline void clamp_sigmoid_and_mul_stub(
    scalar_t* __restrict__ out,
    const scalar_t* __restrict__ input,
    int64_t size,
    const float alpha,
    const float limit) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;
  const fVec one = fVec(1.f);
  const fVec zero = fVec(0.f);
  const fVec limit_v = fVec(limit);
  const fVec nlimit_v = fVec(-limit);
  const fVec alpha_v = fVec(alpha);
// ...
    convert_from_float_and_store<scalar_t>(out + d / 2, x0);
  }
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Top-k selection / Top-k 选择
- Integer kernel specialization / 整数内核专用化
- SIMD vectorization / SIMD 向量化
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
