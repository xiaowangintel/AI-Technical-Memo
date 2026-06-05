# norm.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/norm.cpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: Namespace and shared declarations
```cpp
#include "common.h"
#include "vec.h"

namespace {

// NB: avoid using `at::vec::map<>` on bfloat16 or half
// Llama4TextL2Norm
template <typename scalar_t>
void l2norm_kernel_impl(
    scalar_t* __restrict__ output,
    const scalar_t* __restrict__ input,
    int64_t batch_size,
    int64_t seq_len,
    int64_t hidden_size,
    int64_t input_strideB,
    int64_t input_strideS,
    int64_t output_strideB,
    int64_t output_strideS,
    float eps = 1e-5) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;

  constexpr int kVecSize = bVec::size();
  at::parallel_for(0, batch_size * seq_len, 0, [&](int64_t begin, int64_t end) {
    int64_t bi{0}, si{0};
    data_index_init(begin, bi, batch_size, si, seq_len);
    for (int64_t i = begin; i < end; ++i) {
      // local ptrs
      scalar_t* __restrict__ out_ptr = output + bi * output_strideB + si * output_strideS;
      const scalar_t* __restrict__ input_ptr = input + bi * input_strideB + si * input_strideS;
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 32-60: Runtime integration and dispatch
```cpp
      fVec sum_fvec = fVec(float(0));
      float sum_val = float(0);

      int64_t d;
#pragma GCC unroll 4
      for (d = 0; d <= hidden_size - kVecSize; d += kVecSize) {
        bVec x_bvec = bVec::loadu(input_ptr + d);
        fVec x_fvec0, x_fvec1;
        std::tie(x_fvec0, x_fvec1) = at::vec::convert_to_float(x_bvec);

        sum_fvec += x_fvec0 * x_fvec0;
        sum_fvec += x_fvec1 * x_fvec1;
      }
#pragma GCC unroll 4
      for (; d < hidden_size; ++d) {
        float x_val = static_cast<float>(input_ptr[d]);
        sum_val += x_val * x_val;
      }

      sum_val += vec_reduce_sum(sum_fvec);
      float rsqrt_var = float(1) / std::sqrt(sum_val / hidden_size + eps);
      const fVec scale_fvec = fVec(rsqrt_var);

#pragma GCC unroll 4
      for (d = 0; d <= hidden_size - kVecSize; d += kVecSize) {
        bVec x_bvec = bVec::loadu(input_ptr + d);
        fVec x_fvec0, x_fvec1;
        std::tie(x_fvec0, x_fvec1) = at::vec::convert_to_float(x_bvec);
```
**EN:** This section uses `fVec`, `float`, `loadu` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fVec`、`float`、`loadu`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 61-88: Templates, aliases, and constants
```cpp
        x_fvec0 = x_fvec0 * scale_fvec;
        x_fvec1 = x_fvec1 * scale_fvec;

        bVec out_bvec = convert_from_float_ext<scalar_t>(x_fvec0, x_fvec1);
        out_bvec.store(out_ptr + d);
      }
#pragma GCC unroll 4
      for (; d < hidden_size; ++d) {
        float x_val = static_cast<float>(input_ptr[d]);
        out_ptr[d] = static_cast<scalar_t>(x_val * rsqrt_var);
      }
      // move to the next index
      data_index_step(bi, batch_size, si, seq_len);
    }
  });
}

template <typename scalar_t, typename func_t, typename vec_func_t>
void rmsnorm_kernel_impl(
    scalar_t* __restrict__ output,
    const scalar_t* __restrict__ input,
    const scalar_t* __restrict__ weight,
    int64_t batch_size,
    int64_t seq_len,
    int64_t hidden_size,
    int64_t input_strideB,
    int64_t input_strideS,
    int64_t output_strideB,
```
**EN:** This section defines `store`, `data_index_step`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`store`、`data_index_step`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 89-117: Runtime integration and dispatch
```cpp
    int64_t output_strideS,
    const func_t& f,
    const vec_func_t& vf,
    float eps = 1e-5) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;

  constexpr int kVecSize = bVec::size();
  at::parallel_for(0, batch_size * seq_len, 0, [&](int64_t begin, int64_t end) {
    int64_t bi{0}, si{0};
    data_index_init(begin, bi, batch_size, si, seq_len);
    for (int64_t i = begin; i < end; ++i) {
      // local ptrs
      scalar_t* __restrict__ out_ptr = output + bi * output_strideB + si * output_strideS;
      const scalar_t* __restrict__ input_ptr = input + bi * input_strideB + si * input_strideS;

      fVec sum_fvec = fVec(float(0));
      float sum_val = float(0);

      int64_t d;
#pragma GCC unroll 4
      for (d = 0; d <= hidden_size - kVecSize; d += kVecSize) {
        bVec x_bvec = bVec::loadu(input_ptr + d);
        fVec x_fvec0, x_fvec1;
        std::tie(x_fvec0, x_fvec1) = at::vec::convert_to_float(x_bvec);

        sum_fvec += x_fvec0 * x_fvec0;
        sum_fvec += x_fvec1 * x_fvec1;
      }
```
**EN:** This section uses `at::parallel_for`, `data_index_init`, `fVec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`data_index_init`、`fVec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 118-143: Runtime integration and dispatch
```cpp
#pragma GCC unroll 4
      for (; d < hidden_size; ++d) {
        float x_val = static_cast<float>(input_ptr[d]);
        sum_val += x_val * x_val;
      }

      sum_val += vec_reduce_sum(sum_fvec);
      float rsqrt_var = float(1) / std::sqrt(sum_val / hidden_size + eps);
      const fVec scale_fvec = fVec(rsqrt_var);

#pragma GCC unroll 4
      for (d = 0; d <= hidden_size - kVecSize; d += kVecSize) {
        bVec x_bvec = bVec::loadu(input_ptr + d);
        fVec x_fvec0, x_fvec1;
        std::tie(x_fvec0, x_fvec1) = at::vec::convert_to_float(x_bvec);

        bVec w_bvec = bVec::loadu(weight + d);
        fVec w_fvec0, w_fvec1;
        std::tie(w_fvec0, w_fvec1) = at::vec::convert_to_float(w_bvec);

        x_fvec0 = x_fvec0 * scale_fvec * vf(w_fvec0);
        x_fvec1 = x_fvec1 * scale_fvec * vf(w_fvec1);

        bVec out_bvec = convert_from_float_ext<scalar_t>(x_fvec0, x_fvec1);
        out_bvec.store(out_ptr + d);
      }
```
**EN:** This section uses `vec_reduce_sum`, `float`, `fVec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`vec_reduce_sum`、`float`、`fVec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 144-174: Runtime integration and dispatch
```cpp
#pragma GCC unroll 4
      for (; d < hidden_size; ++d) {
        float x_val = static_cast<float>(input_ptr[d]);
        float w_val = static_cast<float>(weight[d]);
        out_ptr[d] = static_cast<scalar_t>(x_val * rsqrt_var * f(w_val));
      }
      // move to the next index
      data_index_step(bi, batch_size, si, seq_len);
    }
  });
}

template <typename scalar_t>
void gemma3_rmsnorm_kernel_4d_impl(
    scalar_t* __restrict__ output,
    const scalar_t* __restrict__ input,
    const scalar_t* __restrict__ weight,
    int64_t batch_size,
    int64_t num_head,
    int64_t seq_len,
    int64_t hidden_size,
    int64_t input_strideB,
    int64_t input_strideH,
    int64_t input_strideS,
    int64_t output_strideB,
    int64_t output_strideH,
    int64_t output_strideS,
    float eps = 1e-5) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;
```
**EN:** This section uses `gemma3_rmsnorm_kernel_4d_impl`, `f`, `data_index_step` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`gemma3_rmsnorm_kernel_4d_impl`、`f`、`data_index_step`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 175-202: Runtime integration and dispatch
```cpp
  constexpr int kVecSize = bVec::size();
  at::parallel_for(0, batch_size * num_head * seq_len, 0, [&](int64_t begin, int64_t end) {
    int64_t bi{0}, hi{0}, si{0};
    data_index_init(begin, bi, batch_size, hi, num_head, si, seq_len);
    for (int64_t i = begin; i < end; ++i) {
      // local ptrs
      scalar_t* __restrict__ out_ptr = output + bi * output_strideB + hi * output_strideH + si * output_strideS;
      const scalar_t* __restrict__ input_ptr = input + bi * input_strideB + hi * input_strideH + si * input_strideS;

      fVec sum_fvec = fVec(float(0));
      float sum_val = float(0);
      fVec one_fvec = fVec(float(1));

      int64_t d;
#pragma GCC unroll 4
      for (d = 0; d <= hidden_size - kVecSize; d += kVecSize) {
        bVec x_bvec = bVec::loadu(input_ptr + d);
        fVec x_fvec0, x_fvec1;
        std::tie(x_fvec0, x_fvec1) = at::vec::convert_to_float(x_bvec);

        sum_fvec += x_fvec0 * x_fvec0;
        sum_fvec += x_fvec1 * x_fvec1;
      }
#pragma GCC unroll 4
      for (; d < hidden_size; ++d) {
        float x_val = static_cast<float>(input_ptr[d]);
        sum_val += x_val * x_val;
      }
```
**EN:** This section uses `at::parallel_for`, `data_index_init`, `fVec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`data_index_init`、`fVec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 203-229: Runtime integration and dispatch
```cpp

      sum_val += vec_reduce_sum(sum_fvec);
      float rsqrt_var = float(1) / std::sqrt(sum_val / hidden_size + eps);
      const fVec scale_fvec = fVec(rsqrt_var);

#pragma GCC unroll 4
      for (d = 0; d <= hidden_size - kVecSize; d += kVecSize) {
        bVec x_bvec = bVec::loadu(input_ptr + d);
        fVec x_fvec0, x_fvec1;
        std::tie(x_fvec0, x_fvec1) = at::vec::convert_to_float(x_bvec);

        bVec w_bvec = bVec::loadu(weight + d);
        fVec w_fvec0, w_fvec1;
        std::tie(w_fvec0, w_fvec1) = at::vec::convert_to_float(w_bvec);

        x_fvec0 = x_fvec0 * scale_fvec * (w_fvec0 + one_fvec);
        x_fvec1 = x_fvec1 * scale_fvec * (w_fvec1 + one_fvec);

        bVec out_bvec = convert_from_float_ext<scalar_t>(x_fvec0, x_fvec1);
        out_bvec.store(out_ptr + d);
      }
#pragma GCC unroll 4
      for (; d < hidden_size; ++d) {
        float x_val = static_cast<float>(input_ptr[d]);
        float w_val = static_cast<float>(weight[d]);
        out_ptr[d] = static_cast<scalar_t>(x_val * rsqrt_var * (w_val + 1));
      }
```
**EN:** This section uses `vec_reduce_sum`, `float`, `fVec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`vec_reduce_sum`、`float`、`fVec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 230-259: Runtime integration and dispatch
```cpp
      // move to the next index
      data_index_step(bi, batch_size, hi, num_head, si, seq_len);
    }
  });
}

template <typename scalar_t, typename func_t, typename vec_func_t>
void fused_add_rmsnorm_kernel_impl(
    scalar_t* __restrict__ input,
    scalar_t* __restrict__ residual,
    const scalar_t* __restrict__ weight,
    float* __restrict__ buffer,
    int64_t batch_size,
    int64_t seq_len,
    int64_t hidden_size,
    int64_t input_strideB,
    int64_t input_strideS,
    const func_t& f,
    const vec_func_t& vf,
    float eps = 1e-5) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;

  constexpr int kVecSize = bVec::size();
  at::parallel_for(0, batch_size * seq_len, 0, [&](int64_t begin, int64_t end) {
    int64_t bi{0}, si{0};
    data_index_init(begin, bi, batch_size, si, seq_len);
    int tid = at::get_thread_num();
    float* __restrict__ buffer_ptr = buffer + tid * hidden_size;
```
**EN:** This section uses `fused_add_rmsnorm_kernel_impl`, `at::parallel_for`, `data_index_step` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fused_add_rmsnorm_kernel_impl`、`at::parallel_for`、`data_index_step`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 260-287: Runtime integration and dispatch
```cpp
    for (int64_t i = begin; i < end; ++i) {
      // local ptrs
      scalar_t* __restrict__ input_ptr = input + bi * input_strideB + si * input_strideS;
      scalar_t* __restrict__ residual_ptr = residual + i * hidden_size;

      fVec sum_fvec = fVec(float(0));
      float sum_val = float(0);

      int64_t d;
#pragma GCC unroll 4
      for (d = 0; d <= hidden_size - kVecSize; d += kVecSize) {
        bVec x_bvec = bVec::loadu(input_ptr + d);
        fVec x_fvec0, x_fvec1;
        std::tie(x_fvec0, x_fvec1) = at::vec::convert_to_float(x_bvec);

        bVec r_bvec = bVec::loadu(residual_ptr + d);
        fVec r_fvec0, r_fvec1;
        std::tie(r_fvec0, r_fvec1) = at::vec::convert_to_float(r_bvec);

        x_fvec0 += r_fvec0;
        x_fvec1 += r_fvec1;

        bVec out_bvec = convert_from_float_ext<scalar_t>(x_fvec0, x_fvec1);
        out_bvec.store(residual_ptr + d);

        sum_fvec += x_fvec0 * x_fvec0;
        sum_fvec += x_fvec1 * x_fvec1;
```
**EN:** This section uses `fVec`, `float`, `loadu` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fVec`、`float`、`loadu`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 288-315: Runtime integration and dispatch
```cpp
        x_fvec0.store(buffer_ptr + d);
        x_fvec1.store(buffer_ptr + d + fVec::size());
      }
#pragma GCC unroll 4
      for (; d < hidden_size; ++d) {
        float x_val = static_cast<float>(input_ptr[d]);
        float r_val = static_cast<float>(residual_ptr[d]);

        x_val += r_val;
        residual_ptr[d] = static_cast<scalar_t>(x_val);

        sum_val += x_val * x_val;
        buffer_ptr[d] = x_val;
      }

      sum_val += vec_reduce_sum(sum_fvec);
      float rsqrt_var = float(1) / std::sqrt(sum_val / hidden_size + eps);
      const fVec scale_fvec = fVec(rsqrt_var);

#pragma GCC unroll 4
      for (d = 0; d <= hidden_size - kVecSize; d += kVecSize) {
        fVec x_fvec0 = fVec::loadu(buffer_ptr + d);
        fVec x_fvec1 = fVec::loadu(buffer_ptr + d + fVec::size());

        bVec w_bvec = bVec::loadu(weight + d);
        fVec w_fvec0, w_fvec1;
        std::tie(w_fvec0, w_fvec1) = at::vec::convert_to_float(w_bvec);
```
**EN:** This section uses `store`, `vec_reduce_sum`, `float` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`store`、`vec_reduce_sum`、`float`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 316-345: Runtime integration and dispatch
```cpp
        x_fvec0 = x_fvec0 * scale_fvec * vf(w_fvec0);
        x_fvec1 = x_fvec1 * scale_fvec * vf(w_fvec1);
        bVec x_bvec = convert_from_float_ext<scalar_t>(x_fvec0, x_fvec1);
        x_bvec.store(input_ptr + d);
      }
#pragma GCC unroll 4
      for (; d < hidden_size; ++d) {
        float x_val = buffer_ptr[d] * rsqrt_var * static_cast<float>(f(weight[d]));
        input_ptr[d] = x_val;
      }
      // move to the next index
      data_index_step(bi, batch_size, si, seq_len);
    }
  });
}

template <typename scalar_t>
void fused_rmsnorm_gated_kernel_impl(
    scalar_t* __restrict__ output,
    const scalar_t* __restrict__ input,
    const scalar_t* __restrict__ weight,
    const scalar_t* __restrict__ gate,
    int64_t batch_size,
    int64_t hidden_size,
    int64_t input_strideN,
    float eps = 1e-5) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;
  const fVec one = fVec(1.f);
```
**EN:** This section uses `fused_rmsnorm_gated_kernel_impl`, `vf`, `store` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fused_rmsnorm_gated_kernel_impl`、`vf`、`store`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 346-372: Runtime integration and dispatch
```cpp
  constexpr int kVecSize = bVec::size();
  at::parallel_for(0, batch_size, 0, [&](int64_t begin, int64_t end) {
    for (int64_t i = begin; i < end; ++i) {
      // local ptrs
      scalar_t* __restrict__ out_ptr = output + i * hidden_size;
      const scalar_t* __restrict__ input_ptr = input + i * input_strideN;
      const scalar_t* __restrict__ gate_ptr = gate + i * hidden_size;

      fVec sum_fvec = fVec(float(0));
      float sum_val = float(0);

      int64_t d;
#pragma GCC unroll 4
      for (d = 0; d <= hidden_size - kVecSize; d += kVecSize) {
        bVec x_bvec = bVec::loadu(input_ptr + d);
        fVec x_fvec0, x_fvec1;
        std::tie(x_fvec0, x_fvec1) = at::vec::convert_to_float(x_bvec);

        sum_fvec += x_fvec0 * x_fvec0;
        sum_fvec += x_fvec1 * x_fvec1;
      }
#pragma GCC unroll 4
      for (; d < hidden_size; ++d) {
        float x_val = static_cast<float>(input_ptr[d]);
        sum_val += x_val * x_val;
      }
```
**EN:** This section uses `at::parallel_for`, `fVec`, `float` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`fVec`、`float`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 373-398: Runtime integration and dispatch
```cpp
      sum_val += vec_reduce_sum(sum_fvec);
      float rsqrt_var = float(1) / std::sqrt(sum_val / hidden_size + eps);
      const fVec scale_fvec = fVec(rsqrt_var);

#pragma GCC unroll 4
      for (d = 0; d <= hidden_size - kVecSize; d += kVecSize) {
        bVec x_bvec = bVec::loadu(input_ptr + d);
        fVec x_fvec0, x_fvec1;
        std::tie(x_fvec0, x_fvec1) = at::vec::convert_to_float(x_bvec);

        bVec w_bvec = bVec::loadu(weight + d);
        fVec w_fvec0, w_fvec1;
        std::tie(w_fvec0, w_fvec1) = at::vec::convert_to_float(w_bvec);

        bVec g_bvec = bVec::loadu(gate_ptr + d);
        fVec g_fvec0, g_fvec1;
        std::tie(g_fvec0, g_fvec1) = at::vec::convert_to_float(g_bvec);
        g_fvec0 = g_fvec0 / (one + g_fvec0.neg().exp_u20());
        g_fvec1 = g_fvec1 / (one + g_fvec1.neg().exp_u20());

        x_fvec0 = x_fvec0 * scale_fvec * w_fvec0 * g_fvec0;
        x_fvec1 = x_fvec1 * scale_fvec * w_fvec1 * g_fvec1;

        bVec out_bvec = convert_from_float_ext<scalar_t>(x_fvec0, x_fvec1);
        out_bvec.store(out_ptr + d);
      }
```
**EN:** This section uses `vec_reduce_sum`, `float`, `fVec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`vec_reduce_sum`、`float`、`fVec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 399-429: Runtime integration and dispatch
```cpp
#pragma GCC unroll 4
      for (; d < hidden_size; ++d) {
        float x_val = static_cast<float>(input_ptr[d]);
        float w_val = static_cast<float>(weight[d]);
        float g_val = static_cast<float>(gate_ptr[d]);

        out_ptr[d] = static_cast<scalar_t>(x_val * rsqrt_var * w_val * g_val / (1.f + std::exp(-g_val)));
      }
    }
  });
}

}  // anonymous namespace

template <typename scalar_t>
void fused_add_layernorm_kernel_impl(
    scalar_t* __restrict__ output,
    const scalar_t* __restrict__ input,
    scalar_t* __restrict__ residual,
    const scalar_t* __restrict__ weight,
    const scalar_t* __restrict__ bias,
    float* __restrict__ buffer,
    int64_t batch_size,
    int64_t seq_len,
    int64_t hidden_size,
    int64_t input_strideN,
    float eps = 1e-5) {
  using bVec = at::vec::Vectorized<scalar_t>;
  using fVec = at::vec::Vectorized<float>;
  constexpr int kVecSize = bVec::size();
```
**EN:** This section uses `fused_add_layernorm_kernel_impl`, `exp`, `bVec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fused_add_layernorm_kernel_impl`、`exp`、`bVec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 430-459: Runtime integration and dispatch
```cpp
  const bool has_residual{residual != nullptr};
  const bool has_bias{bias != nullptr};
  const int64_t parallel_size{batch_size * seq_len};
  at::parallel_for(0, parallel_size, 0, [&](int64_t begin, int64_t end) {
    float* __restrict__ buffer_ptr = buffer + at::get_thread_num() * hidden_size;

    for (int64_t i = begin; i < end; ++i) {
      scalar_t* __restrict__ out_ptr = output + i * hidden_size;
      const scalar_t* __restrict__ input_ptr = input + i * input_strideN;
      scalar_t* __restrict__ residual_ptr{(scalar_t*)nullptr};
      if (has_residual) {
        residual_ptr = residual + i * hidden_size;
      }

      // First pass: compute mean and var
      fVec sum_fvec{fVec(0.0)}, sum_sq_fvec{fVec(0.0)};
      float sum_val{0.0}, sum_sq_val{0.0};
      int64_t d{0};

#pragma GCC unroll 4
      for (; d <= hidden_size - kVecSize; d += kVecSize) {
        bVec x_bvec = bVec::loadu(input_ptr + d);
        fVec x_fvec0, x_fvec1;
        std::tie(x_fvec0, x_fvec1) = at::vec::convert_to_float(x_bvec);

        if (has_residual) {
          bVec r_bvec = bVec::loadu(residual_ptr + d);
          fVec r_fvec0, r_fvec1;
          std::tie(r_fvec0, r_fvec1) = at::vec::convert_to_float(r_bvec);
```
**EN:** This section uses `at::parallel_for`, `loadu`, `tie` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`loadu`、`tie`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 460-487: Control flow and branching
```cpp
          x_fvec0 += r_fvec0;
          x_fvec1 += r_fvec1;

          bVec out_bvec = convert_from_float_ext<scalar_t>(x_fvec0, x_fvec1);
          out_bvec.store(residual_ptr + d);
        }

        sum_fvec += x_fvec0;
        sum_fvec += x_fvec1;
        sum_sq_fvec += x_fvec0 * x_fvec0;
        sum_sq_fvec += x_fvec1 * x_fvec1;

        x_fvec0.store(buffer_ptr + d);
        x_fvec1.store(buffer_ptr + d + fVec::size());
      }
#pragma GCC unroll 4
      for (; d < hidden_size; ++d) {
        float x_val = static_cast<float>(input_ptr[d]);
        if (has_residual) {
          float r_val = static_cast<float>(residual_ptr[d]);
          x_val += r_val;
          residual_ptr[d] = static_cast<scalar_t>(x_val);
        }

        sum_val += x_val;
        sum_sq_val += x_val * x_val;
        buffer_ptr[d] = x_val;
      }
```
**EN:** This section drives `store` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`store`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 488-514: Runtime integration and dispatch
```cpp

      // Var(X) = E(X^2) - (E(X))^2
      // Refer to FlashInfer impl:
      // https://github.com/flashinfer-ai/flashinfer/blob/6bb01d19c2d9ab3b6a3a5e9e97448891a5ed2844/include/flashinfer/norm.cuh#L554
      sum_val += vec_reduce_sum(sum_fvec);
      sum_sq_val += vec_reduce_sum(sum_sq_fvec);

      float mean{sum_val / hidden_size};
      float mean_sq{sum_sq_val / hidden_size};
      float variance{mean_sq - (mean * mean)};
      float rsqrt_var{float(1) / std::sqrt(variance + eps)};

      const fVec mean_fvec = fVec(mean);
      const fVec scale_fvec = fVec(rsqrt_var);

      // Second pass: apply normalization
#pragma GCC unroll 4
      for (d = 0; d <= hidden_size - kVecSize; d += kVecSize) {
        fVec x_fvec0 = fVec::loadu(buffer_ptr + d);
        fVec x_fvec1 = fVec::loadu(buffer_ptr + d + fVec::size());
        bVec w_bvec = bVec::loadu(weight + d);
        fVec w_fvec0, w_fvec1;
        std::tie(w_fvec0, w_fvec1) = at::vec::convert_to_float(w_bvec);

        x_fvec0 = (x_fvec0 - mean_fvec) * scale_fvec * w_fvec0;
        x_fvec1 = (x_fvec1 - mean_fvec) * scale_fvec * w_fvec1;
```
**EN:** This section uses `vec_reduce_sum`, `fVec`, `loadu` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`vec_reduce_sum`、`fVec`、`loadu`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 515-546: Runtime integration and dispatch
```cpp
        if (has_bias) {
          bVec b_bvec = bVec::loadu(bias + d);
          fVec b_fvec0, b_fvec1;
          std::tie(b_fvec0, b_fvec1) = at::vec::convert_to_float(b_bvec);
          x_fvec0 += b_fvec0;
          x_fvec1 += b_fvec1;
        }

        bVec o_bvec = convert_from_float_ext<scalar_t>(x_fvec0, x_fvec1);
        o_bvec.store(out_ptr + d);
      }
#pragma GCC unroll 4
      for (; d < hidden_size; ++d) {
        float normalized = (buffer_ptr[d] - mean) * rsqrt_var;
        float x_val = normalized * static_cast<float>(weight[d]);
        if (has_bias) {
          x_val += static_cast<float>(bias[d]);
        }
        out_ptr[d] = static_cast<scalar_t>(x_val);
      }
    }
  });
}  // anonymous namespace

// input : {batch_size, hidden_size}
at::Tensor l2norm_cpu(at::Tensor& input, double eps) {
  CHECK_INPUT(input);
  CHECK_DIM(2, input);
  int64_t batch_size = input.size(0);
  int64_t hidden_size = input.size(1);
  at::Tensor output = at::empty_like(input);
```
**EN:** This section uses `l2norm_cpu`, `loadu`, `tie` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`l2norm_cpu`、`loadu`、`tie`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 547-572: Runtime integration and dispatch
```cpp
  AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "l2norm_kernel", [&] {
    l2norm_kernel_impl<scalar_t>(
        output.data_ptr<scalar_t>(),
        input.data_ptr<scalar_t>(),
        batch_size,
        1,
        hidden_size,
        hidden_size,
        0,
        hidden_size,
        0,
        eps);
  });
  return output;
}

// input : {batch_size, hidden_size} or {batch_size, seq_len, hidden_size}
// weight: {hidden_size}
at::Tensor rmsnorm_cpu(at::Tensor& input, at::Tensor& weight, double eps) {
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(input);
  CHECK_INPUT(weight);
  int64_t inp_dim{input.dim()};
  TORCH_CHECK(inp_dim == 2 || inp_dim == 3, "Expected input dim to be 2 or 3, but got ", inp_dim);
  CHECK_DIM(1, weight);
  CHECK_EQ(input.size(-1), weight.size(0));
```
**EN:** This section uses `rmsnorm_cpu`, `CHECK_LAST_DIM_CONTIGUOUS_INPUT`, `CHECK_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`rmsnorm_cpu`、`CHECK_LAST_DIM_CONTIGUOUS_INPUT`、`CHECK_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 573-605: Runtime integration and dispatch
```cpp
  int64_t batch_size = input.size(0);
  int64_t seq_len = 1;
  int64_t hidden_size = input.size(-1);
  int64_t input_strideB = input.stride(0);
  int64_t input_strideS = 0;
  at::Tensor output = at::empty_like(input);
  int64_t output_strideB = output.stride(0);
  int64_t output_strideS = 0;
  if (inp_dim == 3) {
    seq_len = input.size(1);
    input_strideS = input.stride(1);
    output_strideS = output.stride(1);
  }

  AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "rmsnorm_kernel", [&] {
    using Vec = at::vec::Vectorized<float>;
    rmsnorm_kernel_impl<scalar_t>(
        output.data_ptr<scalar_t>(),
        input.data_ptr<scalar_t>(),
        weight.data_ptr<scalar_t>(),
        batch_size,
        seq_len,
        hidden_size,
        input_strideB,
        input_strideS,
        output_strideB,
        output_strideS,
        [](float x) { return x; },
        [](Vec x) { return x; },
        eps);
  });
  return output;
}
```
**EN:** This section uses `stride`, `empty_like`, `Vec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`stride`、`empty_like`、`Vec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 606-635: Runtime integration and dispatch
```cpp

// input : {batch_size, hidden_size} or {batch_size, seq_len, hidden_size}
// weight: {hidden_size}
// bias  : {hidden_size}
at::Tensor
layernorm_cpu(const at::Tensor& input, const at::Tensor& weight, const std::optional<at::Tensor>& bias, double eps) {
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(input);
  CHECK_INPUT(weight);
  int64_t inp_dim{input.dim()};
  TORCH_CHECK(inp_dim == 2 || inp_dim == 3, "Expected input dim to be 2 or 3, but got ", inp_dim);
  CHECK_DIM(1, weight);
  if (bias.has_value()) {
    CHECK_DIM(1, bias.value());
    CHECK_EQ(bias.value().size(0), weight.size(0));
  }

  int64_t batch_size{input.size(0)}, seq_len{1}, hidden_size{input.size(1)}, input_strideN{input.stride(0)};
  if (inp_dim == 3) {
    CHECK_EQ(input.size(2), weight.size(0));
    seq_len = input.size(1);
    hidden_size = input.size(2);
    input_strideN = input.stride(1);
  } else {
    CHECK_EQ(input.size(1), weight.size(0));
  }

  at::Tensor output = at::empty_like(input);
  int64_t num_threads = at::get_num_threads();
  at::Tensor buffer = at::empty({num_threads, hidden_size}, input.options().dtype(at::kFloat));
```
**EN:** This section uses `layernorm_cpu`, `CHECK_LAST_DIM_CONTIGUOUS_INPUT`, `CHECK_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`layernorm_cpu`、`CHECK_LAST_DIM_CONTIGUOUS_INPUT`、`CHECK_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 636-664: Runtime integration and dispatch
```cpp
  AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "layernorm_kernel", [&] {
    fused_add_layernorm_kernel_impl<scalar_t>(
        output.data_ptr<scalar_t>(),
        input.data_ptr<scalar_t>(),
        nullptr,
        weight.data_ptr<scalar_t>(),
        conditional_data_ptr<scalar_t>(bias),
        buffer.data_ptr<float>(),
        batch_size,
        seq_len,
        hidden_size,
        input_strideN,
        eps);
  });
  return output;
}

at::Tensor gemma_rmsnorm_cpu(at::Tensor& input, at::Tensor& weight, double eps) {
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(input);
  CHECK_INPUT(weight);
  CHECK_DIM(2, input);
  CHECK_DIM(1, weight);
  CHECK_EQ(input.size(1), weight.size(0));
  int64_t batch_size = input.size(0);
  int64_t hidden_size = input.size(1);
  at::Tensor output = at::empty_like(input);
  int64_t input_strideN = input.stride(0);
  int64_t output_strideN = output.stride(0);
```
**EN:** This section uses `gemma_rmsnorm_cpu`, `CHECK_LAST_DIM_CONTIGUOUS_INPUT`, `CHECK_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`gemma_rmsnorm_cpu`、`CHECK_LAST_DIM_CONTIGUOUS_INPUT`、`CHECK_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 665-692: Runtime integration and dispatch
```cpp
  AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "gemma_rmsnorm_kernel", [&] {
    using Vec = at::vec::Vectorized<float>;
    Vec one_vec = Vec(float(1));
    rmsnorm_kernel_impl<scalar_t>(
        output.data_ptr<scalar_t>(),
        input.data_ptr<scalar_t>(),
        weight.data_ptr<scalar_t>(),
        batch_size,
        1,
        hidden_size,
        input_strideN,
        0,
        output_strideN,
        0,
        [](float x) { return x + 1; },
        [one_vec](Vec x) { return x + one_vec; },
        eps);
  });
  return output;
}

// input : {batch_size, hidden_size} or {batch_size, num_head, seq_len, head_dim}
// weight: {hidden_size}
at::Tensor gemma3_rmsnorm_cpu(at::Tensor& input, at::Tensor& weight, double eps) {
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(input);
  CHECK_INPUT(weight);
  TORCH_CHECK(
      input.dim() == 2 || input.dim() == 4, "gemma3_rmsnorm_cpu: input must be 2D or 4D, got ", input.dim(), "D");
```
**EN:** This section uses `gemma3_rmsnorm_cpu`, `Vec`, `CHECK_LAST_DIM_CONTIGUOUS_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`gemma3_rmsnorm_cpu`、`Vec`、`CHECK_LAST_DIM_CONTIGUOUS_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 693-720: Runtime integration and dispatch
```cpp
  CHECK_DIM(1, weight);
  CHECK_EQ(input.size(-1), weight.size(0));
  int64_t batch_size = input.size(0);
  int64_t hidden_size = weight.size(0);
  at::Tensor output = at::empty_like(input);
  if (input.dim() == 2) {
    int64_t input_strideN = input.stride(0);
    int64_t output_strideN = output.stride(0);

    AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "gemma3_rmsnorm_kernel", [&] {
      using Vec = at::vec::Vectorized<float>;
      Vec one_vec = Vec(float(1));
      rmsnorm_kernel_impl<scalar_t>(
          output.data_ptr<scalar_t>(),
          input.data_ptr<scalar_t>(),
          weight.data_ptr<scalar_t>(),
          batch_size,
          1,
          hidden_size,
          input_strideN,
          0,
          output_strideN,
          0,
          [](float x) { return x + 1; },
          [one_vec](Vec x) { return x + one_vec; },
          eps);
    });
  } else {
```
**EN:** This section uses `CHECK_DIM`, `CHECK_EQ`, `empty_like` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_DIM`、`CHECK_EQ`、`empty_like`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 721-748: Local implementation details
```cpp
    int64_t input_strideB = input.stride(0);
    int64_t input_strideH = input.stride(1);
    int64_t input_strideS = input.stride(2);
    int64_t output_strideB = output.stride(0);
    int64_t output_strideH = output.stride(1);
    int64_t output_strideS = output.stride(2);
    int64_t num_head = input.size(1);
    int64_t seq_len = input.size(2);
    AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "gemma3_rmsnorm_kernel", [&] {
      gemma3_rmsnorm_kernel_4d_impl<scalar_t>(
          output.data_ptr<scalar_t>(),
          input.data_ptr<scalar_t>(),
          weight.data_ptr<scalar_t>(),
          batch_size,
          num_head,
          seq_len,
          hidden_size,
          input_strideB,
          input_strideH,
          input_strideS,
          output_strideB,
          output_strideH,
          output_strideS,
          eps);
    });
  }
  return output;
}
```
**EN:** This section fills in the local implementation details around `stride`, completing the behavior required by the file.
**CN:** 本段补充了`stride`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 749-774: Runtime integration and dispatch
```cpp

// Gemma4RMSNorm: with_scale ? norm(x) * (weight + scale_shift) : norm(x)
// input : {batch_size, hidden_size} or {batch_size, seq_len, hidden_size}
// weight: {hidden_size}
at::Tensor gemma4_rmsnorm_cpu(at::Tensor& input, at::Tensor& weight, double eps, double scale_shift, bool with_scale) {
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(input);
  CHECK_INPUT(weight);
  int64_t inp_dim{input.dim()};
  TORCH_CHECK(inp_dim == 2 || inp_dim == 3, "gemma4_rmsnorm_cpu: expected input dim 2 or 3, got ", inp_dim);
  CHECK_DIM(1, weight);
  CHECK_EQ(input.size(-1), weight.size(0));

  int64_t hidden_size = input.size(-1);
  at::Tensor output = at::empty_like(input);
  int64_t batch_size = input.size(0);
  int64_t seq_len = 1;
  int64_t input_strideB = input.stride(0);
  int64_t input_strideS = 0;
  int64_t output_strideB = output.stride(0);
  int64_t output_strideS = 0;
  if (inp_dim == 3) {
    seq_len = input.size(1);
    input_strideS = input.stride(1);
    output_strideS = output.stride(1);
  }
```
**EN:** This section uses `gemma4_rmsnorm_cpu`, `CHECK_LAST_DIM_CONTIGUOUS_INPUT`, `CHECK_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`gemma4_rmsnorm_cpu`、`CHECK_LAST_DIM_CONTIGUOUS_INPUT`、`CHECK_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 775-802: Runtime integration and dispatch
```cpp
  if (with_scale) {
    float shift = static_cast<float>(scale_shift);
    AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "gemma4_rmsnorm_kernel", [&] {
      using Vec = at::vec::Vectorized<float>;
      Vec shift_vec = Vec(shift);
      rmsnorm_kernel_impl<scalar_t>(
          output.data_ptr<scalar_t>(),
          input.data_ptr<scalar_t>(),
          weight.data_ptr<scalar_t>(),
          batch_size,
          seq_len,
          hidden_size,
          input_strideB,
          input_strideS,
          output_strideB,
          output_strideS,
          [shift](float x) { return x + shift; },
          [shift_vec](Vec x) { return x + shift_vec; },
          eps);
    });
  } else {
    AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "gemma4_rmsnorm_kernel", [&] {
      l2norm_kernel_impl<scalar_t>(
          output.data_ptr<scalar_t>(),
          input.data_ptr<scalar_t>(),
          batch_size,
          seq_len,
          hidden_size,
```
**EN:** This section uses `Vec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`Vec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 803-830: Runtime integration and dispatch
```cpp
          input_strideB,
          input_strideS,
          output_strideB,
          output_strideS,
          eps);
    });
  }
  return output;
}

// input : {batch_size, hidden_size}
// weight: {hidden_size}
// gate: {batch_size, hidden_size}
at::Tensor fused_rmsnorm_gated_cpu(at::Tensor& input, at::Tensor& weight, at::Tensor& gate, double eps) {
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(input);
  CHECK_INPUT(weight);
  CHECK_INPUT(gate);
  CHECK_DIM(2, input);
  CHECK_DIM(1, weight);
  CHECK_DIM(2, gate);
  CHECK_EQ(input.size(1), weight.size(0));
  int64_t batch_size = input.size(0);
  int64_t hidden_size = input.size(1);
  CHECK_EQ(input.size(0), gate.size(0));
  CHECK_EQ(input.size(1), gate.size(1));
  at::Tensor output = at::empty_like(input);
  int64_t input_strideN = input.stride(0);
```
**EN:** This section uses `fused_rmsnorm_gated_cpu`, `CHECK_LAST_DIM_CONTIGUOUS_INPUT`, `CHECK_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fused_rmsnorm_gated_cpu`、`CHECK_LAST_DIM_CONTIGUOUS_INPUT`、`CHECK_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 831-859: Runtime integration and dispatch
```cpp
  AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "fused_rmsnorm_gated_kernel", [&] {
    fused_rmsnorm_gated_kernel_impl<scalar_t>(
        output.data_ptr<scalar_t>(),
        input.data_ptr<scalar_t>(),
        weight.data_ptr<scalar_t>(),
        gate.data_ptr<scalar_t>(),
        batch_size,
        hidden_size,
        input_strideN,
        eps);
  });
  return output;
}

// input   : {batch_size, hidden_size} or {batch_size, seq_len, hidden_size}
// residual: {batch_size, hidden_size} or {batch_size, seq_len, hidden_size}
// weight  : {hidden_size}
void fused_add_rmsnorm_cpu(at::Tensor& input, at::Tensor& residual, at::Tensor& weight, double eps) {
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(input);
  CHECK_INPUT(residual);
  CHECK_INPUT(weight);
  int64_t inp_dim{input.dim()}, res_dim{residual.dim()};
  CHECK_EQ(inp_dim, res_dim);
  TORCH_CHECK(inp_dim == 2 || inp_dim == 3, "Expected input dim to be 2 or 3, but got ", inp_dim);
  CHECK_DIM(1, weight);
  CHECK_EQ(input.size(0), residual.size(0));
  CHECK_EQ(input.size(-1), residual.size(-1));
  CHECK_EQ(input.size(-1), weight.size(0));
```
**EN:** This section uses `fused_add_rmsnorm_cpu`, `CHECK_LAST_DIM_CONTIGUOUS_INPUT`, `CHECK_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fused_add_rmsnorm_cpu`、`CHECK_LAST_DIM_CONTIGUOUS_INPUT`、`CHECK_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 860-891: Runtime integration and dispatch
```cpp
  int64_t batch_size = input.size(0);
  int64_t seq_len = 1;
  int64_t hidden_size = input.size(-1);
  int64_t input_strideB = input.stride(0);
  int64_t input_strideS = 0;
  if (inp_dim == 3) {
    seq_len = input.size(1);
    input_strideS = input.stride(1);
  }

  // allocate temp buffer to store x in float32 per thread
  // TODO: implement a singleton for context
  int64_t num_threads = at::get_num_threads();
  at::Tensor buffer = at::empty({num_threads, hidden_size}, input.options().dtype(at::kFloat));

  AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "fused_add_rmsnorm_kernel", [&] {
    using Vec = at::vec::Vectorized<float>;
    fused_add_rmsnorm_kernel_impl<scalar_t>(
        input.data_ptr<scalar_t>(),
        residual.data_ptr<scalar_t>(),
        weight.data_ptr<scalar_t>(),
        buffer.data_ptr<float>(),
        batch_size,
        seq_len,
        hidden_size,
        input_strideB,
        input_strideS,
        [](float x) { return x; },
        [](Vec x) { return x; },
        eps);
  });
}
```
**EN:** This section uses `stride`, `get_num_threads`, `options` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`stride`、`get_num_threads`、`options`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 892-914: Runtime integration and dispatch
```cpp

// input   : {batch_size, hidden_size}
// residual: {batch_size, hidden_size}
// weight  : {hidden_size}
void gemma_fused_add_rmsnorm_cpu(at::Tensor& input, at::Tensor& residual, at::Tensor& weight, double eps) {
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(input);
  CHECK_INPUT(residual);
  CHECK_INPUT(weight);
  CHECK_DIM(2, input);
  CHECK_DIM(2, residual);
  CHECK_DIM(1, weight);
  CHECK_EQ(input.size(0), residual.size(0));
  CHECK_EQ(input.size(1), residual.size(1));
  CHECK_EQ(input.size(1), weight.size(0));
  int64_t batch_size = input.size(0);
  int64_t hidden_size = input.size(1);
  int64_t input_strideN = input.stride(0);

  // allocate temp buffer to store x in float32 per thread
  // TODO: implement a singleton for context
  int64_t num_threads = at::get_num_threads();
  at::Tensor buffer = at::empty({num_threads, hidden_size}, input.options().dtype(at::kFloat));
```
**EN:** This section uses `gemma_fused_add_rmsnorm_cpu`, `CHECK_LAST_DIM_CONTIGUOUS_INPUT`, `CHECK_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`gemma_fused_add_rmsnorm_cpu`、`CHECK_LAST_DIM_CONTIGUOUS_INPUT`、`CHECK_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 915-942: Runtime integration and dispatch
```cpp
  AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "gemma_fused_add_rmsnorm_kernel", [&] {
    using Vec = at::vec::Vectorized<float>;
    Vec one_vec = Vec(float(1));
    fused_add_rmsnorm_kernel_impl<scalar_t>(
        input.data_ptr<scalar_t>(),
        residual.data_ptr<scalar_t>(),
        weight.data_ptr<scalar_t>(),
        buffer.data_ptr<float>(),
        batch_size,
        1,
        hidden_size,
        input_strideN,
        0,
        [](float x) { return x + 1; },
        [one_vec](Vec x) { return x + one_vec; },
        eps);
  });
}

// input   : {batch_size, hidden_size} or {batch_size, seq_len, hidden_size}
// residual: {batch_size, hidden_size} or {batch_size, seq_len, hidden_size}
// weight  : {hidden_size}
// bias    : {hidden_size}
at::Tensor fused_add_layernorm_cpu(
    const at::Tensor& input,
    at::Tensor& residual,
    const at::Tensor& weight,
    const std::optional<at::Tensor>& bias,
```
**EN:** This section uses `Vec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`Vec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 943-971: Runtime integration and dispatch
```cpp
    double eps) {
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(input);
  CHECK_INPUT(residual);
  CHECK_INPUT(weight);
  int64_t inp_dim{input.dim()}, res_dim{residual.dim()};
  CHECK_EQ(inp_dim, res_dim);
  TORCH_CHECK(inp_dim == 2 || inp_dim == 3, "Expected input dim to be 2 or 3, but got ", inp_dim);
  TORCH_CHECK(res_dim == 2 || res_dim == 3, "Expected residual dim to be 2 or 3, but got ", res_dim);

  CHECK_DIM(1, weight);
  if (bias.has_value()) {
    CHECK_DIM(1, bias.value());
    CHECK_EQ(bias.value().size(0), weight.size(0));
  }
  CHECK_EQ(input.size(0), residual.size(0));
  CHECK_EQ(input.size(1), residual.size(1));
  if (inp_dim == 3) {
    CHECK_EQ(input.size(2), residual.size(2));
    CHECK_EQ(input.size(2), weight.size(0));
  } else {
    CHECK_EQ(input.size(1), weight.size(0));
  }

  int64_t batch_size{input.size(0)}, seq_len{1}, hidden_size{input.size(1)}, input_strideN{input.stride(0)};
  if (inp_dim == 3) {
    seq_len = input.size(1);
    hidden_size = input.size(2);
    input_strideN = input.stride(1);
  }
```
**EN:** This section uses `CHECK_LAST_DIM_CONTIGUOUS_INPUT`, `CHECK_INPUT`, `CHECK_EQ` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_LAST_DIM_CONTIGUOUS_INPUT`、`CHECK_INPUT`、`CHECK_EQ`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 972-994: Runtime integration and dispatch
```cpp
  at::Tensor output = at::empty_like(input);

  // Allocate temp buffer to store x in float32 per thread
  // It is necessary to store FP32 precision of residual-add results to pass UT acc test
  int64_t num_threads = at::get_num_threads();
  at::Tensor buffer = at::empty({num_threads, hidden_size}, input.options().dtype(at::kFloat));

  AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "fused_add_layernorm_kernel", [&] {
    fused_add_layernorm_kernel_impl<scalar_t>(
        output.data_ptr<scalar_t>(),
        input.data_ptr<scalar_t>(),
        residual.data_ptr<scalar_t>(),
        weight.data_ptr<scalar_t>(),
        conditional_data_ptr<scalar_t>(bias),
        buffer.data_ptr<float>(),
        batch_size,
        seq_len,
        hidden_size,
        input_strideN,
        eps);
  });
  return output;
}
```
**EN:** This section uses `empty_like`, `get_num_threads`, `options` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`empty_like`、`get_num_threads`、`options`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **Validation / 校验测试**: Contains checks or scaffolding for correctness verification. / 包含正确性验证所需的检查或脚手架。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `common.h`, `vec.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cpu / norm.cpp
