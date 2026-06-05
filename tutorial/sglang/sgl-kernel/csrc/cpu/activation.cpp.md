# activation.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/activation.cpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Namespace and shared declarations
```cpp
#include "common.h"
#include "vec.h"

namespace {

template <typename scalar_t, typename func_t, typename vec_func_t>
void act_and_mul_kernel_impl(
    scalar_t* __restrict__ output,
    const scalar_t* __restrict__ input,
    int64_t num_tokens,
    int64_t dim,
    const func_t& f,
    const vec_func_t& vf) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;

  constexpr int64_t kVecSize = bVec::size();
  at::parallel_for(0, num_tokens, 0, [&](int64_t begin, int64_t end) {
    for (int64_t i = begin; i < end; ++i) {
      // local ptrs
      const scalar_t* __restrict__ input_ptr = input + i * 2 * dim;
      const scalar_t* __restrict__ input_other_ptr = input_ptr + dim;
      scalar_t* __restrict__ output_ptr = output + i * dim;
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 25-44: Runtime integration and dispatch
```cpp
      int64_t d;
#pragma GCC unroll 4
      for (d = 0; d <= dim - kVecSize; d += kVecSize) {
        bVec x_bvec = bVec::loadu(input_ptr + d);
        fVec x_fvec0, x_fvec1;
        std::tie(x_fvec0, x_fvec1) = at::vec::convert_to_float(x_bvec);

        bVec y_bvec = bVec::loadu(input_other_ptr + d);
        fVec y_fvec0, y_fvec1;
        std::tie(y_fvec0, y_fvec1) = at::vec::convert_to_float(y_bvec);

        x_fvec0 = vf(x_fvec0);
        x_fvec1 = vf(x_fvec1);

        x_fvec0 = x_fvec0 * y_fvec0;
        x_fvec1 = x_fvec1 * y_fvec1;

        x_bvec = convert_from_float_ext<scalar_t>(x_fvec0, x_fvec1);
        x_bvec.store(output_ptr + d);
      }
```
**EN:** This section uses `loadu`, `tie`, `vf` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`loadu`、`tie`、`vf`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 45-66: Runtime integration and dispatch
```cpp
#pragma GCC unroll 4
      for (; d < dim; ++d) {
        float x_val = static_cast<float>(input_ptr[d]);
        float y_val = static_cast<float>(input_other_ptr[d]);
        output_ptr[d] = f(x_val) * y_val;
      }
    }
  });
}

}  // anonymous namespace

// input   : {num_tokens, 2 * d}
// output  : {num_tokens, d}
at::Tensor silu_and_mul_cpu(at::Tensor& input) {
  auto sizes = input.sizes().vec();
  int64_t last_dim = input.ndimension() - 1;
  int64_t d = sizes[last_dim] / 2;
  sizes[last_dim] = d;
  int64_t num_tokens = input.numel() / input.size(-1);
  at::Tensor out = at::empty(sizes, input.options());
```
**EN:** This section uses `silu_and_mul_cpu`, `sizes`, `numel` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`silu_and_mul_cpu`、`sizes`、`numel`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 67-88: Runtime integration and dispatch
```cpp
  AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "silu_and_mul", [&] {
    using Vec = at::vec::Vectorized<float>;
    act_and_mul_kernel_impl(
        out.data_ptr<scalar_t>(),
        input.data_ptr<scalar_t>(),
        num_tokens,
        d,
        [](float x) { return x / (1.f + std::exp(-x)); },
        [](Vec x) { return x / (Vec(1.f) + x.neg().exp()); });
  });
  return out;
}

at::Tensor gelu_tanh_and_mul_cpu(const at::Tensor& input) {
  auto sizes = input.sizes().vec();
  int64_t last_dim = input.ndimension() - 1;
  int64_t d = sizes[last_dim] / 2;
  sizes[last_dim] = d;
  int64_t num_tokens = input.numel() / input.size(-1);
  at::Tensor out = at::empty(sizes, input.options());
  const float sqrt_2_div_pi = std::sqrt(2.f / M_PI);
```
**EN:** This section uses `act_and_mul_kernel_impl`, `gelu_tanh_and_mul_cpu`, `exp` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`act_and_mul_kernel_impl`、`gelu_tanh_and_mul_cpu`、`exp`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 89-110: Runtime integration and dispatch
```cpp
  AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "gelu_tanh_and_mul", [&] {
    using Vec = at::vec::Vectorized<float>;
    act_and_mul_kernel_impl(
        out.data_ptr<scalar_t>(),
        input.data_ptr<scalar_t>(),
        num_tokens,
        d,
        [sqrt_2_div_pi](float x) {
          float x3 = x * x * x;
          float tanh_arg = sqrt_2_div_pi * (x + 0.044715f * x3);
          return 0.5f * x * (1.f + std::tanh(tanh_arg));
        },
        [sqrt_2_div_pi](Vec x) {
          Vec x3 = x * x * x;
          Vec tanh_arg = Vec(sqrt_2_div_pi) * (x + Vec(0.044715f) * x3);
          return Vec(0.5f) * x * (Vec(1.f) + tanh_arg.tanh());
        });
  });

  return out;
}
```
**EN:** This section uses `act_and_mul_kernel_impl`, `tanh`, `Vec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`act_and_mul_kernel_impl`、`tanh`、`Vec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 111-132: Runtime integration and dispatch
```cpp
at::Tensor gelu_and_mul_cpu(const at::Tensor& input) {
  auto sizes = input.sizes().vec();
  int64_t last_dim = input.ndimension() - 1;
  int64_t d = sizes[last_dim] / 2;
  sizes[last_dim] = d;
  int64_t num_tokens = input.numel() / input.size(-1);
  at::Tensor out = at::empty(sizes, input.options());

  AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "gelu_and_mul", [&] {
    using Vec = at::vec::Vectorized<float>;
    const float inv_sqrt2 = 1.0f / std::sqrt(2.0f);
    act_and_mul_kernel_impl(
        out.data_ptr<scalar_t>(),
        input.data_ptr<scalar_t>(),
        num_tokens,
        d,
        [inv_sqrt2](float x) { return 0.5f * x * (1.f + std::erf(x * inv_sqrt2)); },
        [inv_sqrt2](Vec x) { return Vec(0.5f) * x * (Vec(1.f) + (x * Vec(inv_sqrt2)).erf()); });
  });

  return out;
}
```
**EN:** This section uses `gelu_and_mul_cpu`, `act_and_mul_kernel_impl`, `sizes` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`gelu_and_mul_cpu`、`act_and_mul_kernel_impl`、`sizes`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `common.h`, `vec.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cpu / activation.cpp
