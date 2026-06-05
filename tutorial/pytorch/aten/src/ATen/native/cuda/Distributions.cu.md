# Distributions.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/Distributions.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `poisson_cuda_kernel`, `curand_uniform_wrapper`, `binomial_cuda_kernel`, `gamma_cuda_kernel`.
- 用途（中文）: 实现与 `poisson_cuda_kernel`, `curand_uniform_wrapper`, `binomial_cuda_kernel`, `gamma_cuda_kernel` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_NO_OPERATORS
   2: #include <ATen/native/cuda/Distributions.h>
   3: #include <ATen/Dispatch.h>
   4: #include <ATen/cuda/CUDAApplyUtils.cuh>
   5: #include <ATen/AccumulateType.h>
   6: #include <ATen/cuda/CUDAGeneratorImpl.h>
   7: #include <ATen/native/UnaryOps.h>
   8: #include <ATen/native/cuda/DistributionTemplates.h>
   9: 
  10: #include <curand.h>
  11: #include <curand_kernel.h>
  12: #include <curand_philox4x32_x.h>
  13: #include <utility>
  14: #include <functional>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/Distributions.h>`, `<ATen/Dispatch.h>`, `<ATen/cuda/CUDAApplyUtils.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/Distributions.h>`, `<ATen/Dispatch.h>`, `<ATen/cuda/CUDAApplyUtils.cuh>`。

### Lines 15-23
```cpp
  15: 
  16: #include <ATen/native/Distributions.h>
  17: #include <ATen/native/cuda/Loops.cuh>
  18: #include <ATen/native/TensorIterator.h>
  19: 
  20: #include <cstdint>
  21: #include <limits>
  22: #include <utility>
  23: #include <type_traits>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/Distributions.h>`, `<ATen/native/cuda/Loops.cuh>`, `<ATen/native/TensorIterator.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/Distributions.h>`, `<ATen/native/cuda/Loops.cuh>`, `<ATen/native/TensorIterator.h>`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 25-39
```cpp
  25: /**
  26:  * Note [Register spilling in curand call for CUDA < 10]
  27:  * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  28:  * For CUDA < 10, curandStatePhilox4_32_10_t engine achieves poor performance (60% SOL bandwidth)
  29:  * when called to generate one random number at a time. This is because the line
  30:  *            unsigned ret = (&state->output.x)[state->STATE++];
  31:  * in
  32:  *            QUALIFIERS unsigned int curand(curandStatePhilox4_32_10_t *state)
  33:  * in curand_kernel.h dynamically indexes into state.output, preventing the compiler from ever
  34:  * storing state.output in registers.
  35:  *
  36:  * CUDA 10 fixed this problem. However, for backwards compatibility, in the following kernels
  37:  * we are using curand distributions that utilize curand4 call. curand4 call doesn't have the
  38:  * register spilling problem.
  39:  */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 41-62
```cpp
  41: namespace {
  42: 
  43: template <typename scalar_t>
  44: void poisson_cuda_kernel(
  45:     const at::TensorBase &ret,
  46:     const at::TensorBase &lambda,
  47:     at::PhiloxCudaState philox_args) {
  48:   auto functor = [philox_args] __device__(
  49:           scalar_t & ret_val, const scalar_t& lambda) {
  50:         CUDA_KERNEL_ASSERT(lambda >= 0 && "invalid Poisson rate, expected rate to be non-negative");
  51:         auto seeds = at::cuda::philox::unpack(philox_args);
  52:         curandStatePhilox4_32_10_t state;
  53:         curand_init(std::get<0>(seeds),
  54:                     blockIdx.x * blockDim.x + threadIdx.x,
  55:                     std::get<1>(seeds),
  56:                     &state);
  57:         ret_val = static_cast<scalar_t>(curand_poisson(&state, lambda));
  58:       };
  59:   at::cuda::CUDA_tensor_apply2<scalar_t, scalar_t, decltype(functor),
  60:                                /*max_threads_per_block=*/512,
  61:                                /*min_blocks_per_sm==*/2>(ret, lambda, functor);
  62: }
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `poisson_cuda_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `poisson_cuda_kernel`, `__device__`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 64-74
```cpp
  64: struct curand_uniform_wrapper {
  65:   curandStatePhilox4_32_10_t &state;
  66:   __device__ curand_uniform_wrapper(curandStatePhilox4_32_10_t &state): state(state) {}
  67:   __device__ float operator()() {
  68: 
  69:   uint32_t val = curand(&state); //need just bits
  70:   constexpr auto MASK = static_cast<uint32_t>((static_cast<uint64_t>(1) << std::numeric_limits<float>::digits) - 1);
  71:   constexpr auto DIVISOR = static_cast<float>(1) / (static_cast<uint32_t>(1) << std::numeric_limits<float>::digits);
  72:     return (val & MASK) * DIVISOR;
  73:   }
  74: };
```
- EN: This block defines or continues the implementation of `curand_uniform_wrapper`.
- CN: 该代码块定义或继续实现 `curand_uniform_wrapper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 76-90
```cpp
  76: template <typename scalar_t>
  77: void binomial_cuda_kernel(
  78:     at::TensorIteratorBase &iter,
  79:     at::PhiloxCudaState philox_args) {
  80:   using accscalar_t = at::acc_type<scalar_t, true>;
  81: 
  82:   at::native::distribution_binary_kernel(iter, philox_args,
  83:       [] GPU_LAMBDA (curandStatePhilox4_32_10_t& state, scalar_t count, scalar_t prob) {
  84:         auto uniform_lambda = curand_uniform_wrapper(state);
  85:         BaseSampler<accscalar_t, decltype(uniform_lambda)> standard_uniform(uniform_lambda);
  86:         auto sample = sample_binomial<scalar_t, accscalar_t, decltype(uniform_lambda)>(count, prob, standard_uniform);
  87:         return static_cast<scalar_t>(sample);
  88:       }
  89:   );
  90: }
```
- EN: This block defines or continues the implementation of `binomial_cuda_kernel`, `distribution_binary_kernel`.
- CN: 该代码块定义或继续实现 `binomial_cuda_kernel`, `distribution_binary_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 92-113
```cpp
  92: template <typename scalar_t>
  93: void gamma_cuda_kernel(
  94:     const at::TensorBase &ret,
  95:     const at::TensorBase &alpha,
  96:     at::PhiloxCudaState philox_args) {
  97:   using accscalar_t = at::acc_type<scalar_t, true>;
  98:   auto functor = [philox_args] __device__(
  99:           scalar_t & ret_val, const scalar_t& alpha) {
 100:         auto seeds = at::cuda::philox::unpack(philox_args);
 101:         curandStatePhilox4_32_10_t state;
 102:         curand_init(std::get<0>(seeds),
 103:                     blockIdx.x * blockDim.x + threadIdx.x,
 104:                     std::get<1>(seeds),
 105:                     &state);
 106: 
 107:         auto uniform_lambda = [&state] __device__ () {
 108:           return curand_uniform(&state);
 109:         };
 110:         BaseSampler<accscalar_t, decltype(uniform_lambda)> standard_uniform(uniform_lambda);
 111: 
 112:         auto normal_lambda = [&state] __device__ () {
 113:           return curand_normal(&state);
```
- EN: This block defines or continues the implementation of `gamma_cuda_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `gamma_cuda_kernel`, `__device__`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 114-123
```cpp
 114:         };
 115:         BaseSampler<accscalar_t, decltype(normal_lambda)> standard_normal(normal_lambda);
 116:         auto sample = sample_gamma<scalar_t, accscalar_t, decltype(uniform_lambda), decltype(normal_lambda)>(alpha, standard_uniform, standard_normal);
 117:         auto min_value = std::numeric_limits<scalar_t>::min();
 118:         ret_val = (min_value > sample) ? min_value : sample;
 119:       };
 120:   at::cuda::CUDA_tensor_apply2<scalar_t, scalar_t, decltype(functor),
 121:                                /*max_threads_per_block=*/256,
 122:                                /*min_blocks_per_sm==*/2>(ret, alpha, functor);
 123: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 125-125
```cpp
 125: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 127-148
```cpp
 127: namespace at::native {
 128: 
 129: void launch_dirichlet_kernel(at::TensorIteratorBase &iter) {
 130:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16,
 131:                                   iter.input_dtype(), "dirichlet_cuda", [&] {
 132:     at::native::gpu_kernel(
 133:         iter,
 134:         [] GPU_LAMBDA (scalar_t gamma, scalar_t gamma_sum) {
 135:       auto ret_val = gamma / gamma_sum;
 136:       auto min_value = std::numeric_limits<scalar_t>::min();
 137:       auto max_value = 1 - std::numeric_limits<scalar_t>::epsilon();
 138:       ret_val = (min_value > ret_val) ? min_value : ret_val;
 139:       ret_val = (max_value < ret_val) ? max_value : ret_val;
 140:       return ret_val;
 141:     });
 142:   });
 143: }
 144: 
 145: void launch_poisson_cuda_kernel(
 146:     const TensorBase &ret, const TensorBase &lambda, CUDAGeneratorImpl *gen) {
 147:   PhiloxCudaState rng_engine_inputs;
 148:   {
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `launch_dirichlet_kernel`, `gpu_kernel`, `launch_poisson_cuda_kernel`.
- CN: 该代码块定义或继续实现 `launch_dirichlet_kernel`, `gpu_kernel`, `launch_poisson_cuda_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 149-149
```cpp
 149:     // See Note [Acquire lock when using random generators]
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 150-156
```cpp
 150:     std::lock_guard<std::mutex> lock(gen->mutex_);
 151:     rng_engine_inputs = gen->philox_cuda_state(20);
 152:   }
 153:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, ret.scalar_type(), "poisson_cuda", [&] {
 154:     poisson_cuda_kernel<scalar_t>(ret, lambda, rng_engine_inputs);
 155:   });
 156: }
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 158-169
```cpp
 158: void launch_binomial_cuda_kernel(
 159:     TensorIteratorBase &iter, CUDAGeneratorImpl *gen) {
 160:   PhiloxCudaState rng_engine_inputs;
 161:   {
 162:     // See Note [Acquire lock when using random generators]
 163:     std::lock_guard<std::mutex> lock(gen->mutex_);
 164:     rng_engine_inputs = gen->philox_cuda_state(42);
 165:   }
 166:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, iter.input_dtype(), "binomial_cuda", [&] {
 167:     binomial_cuda_kernel<scalar_t>(iter, rng_engine_inputs);
 168:   });
 169: }
```
- EN: This block defines or continues the implementation of `launch_binomial_cuda_kernel`.
- CN: 该代码块定义或继续实现 `launch_binomial_cuda_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 171-182
```cpp
 171: void launch_gamma_kernel(
 172:     const TensorBase &ret, const TensorBase &alpha, CUDAGeneratorImpl *gen) {
 173:   PhiloxCudaState rng_engine_inputs;
 174:   {
 175:     // See Note [Acquire lock when using random generators]
 176:     std::lock_guard<std::mutex> lock(gen->mutex_);
 177:     rng_engine_inputs = gen->philox_cuda_state(10);
 178:   }
 179:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, ret.scalar_type(), "gamma_cuda", [&] {
 180:      gamma_cuda_kernel<scalar_t>(ret, alpha, rng_engine_inputs);
 181:    });
 182: }
```
- EN: This block defines or continues the implementation of `launch_gamma_kernel`.
- CN: 该代码块定义或继续实现 `launch_gamma_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 184-192
```cpp
 184: void launch_standard_gamma_grad_kernel(TensorIteratorBase &iter) {
 185:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, iter.input_dtype(), "_standard_gamma_grad_cuda", [&] {
 186:     using accscalar_t = at::acc_type<scalar_t, true>;
 187:     gpu_kernel(iter,
 188:       [] GPU_LAMBDA (scalar_t self_val, scalar_t output_val) {
 189:         return standard_gamma_grad_one<scalar_t, accscalar_t>(self_val, output_val);
 190:       });
 191:   });
 192: }
```
- EN: This block defines or continues the implementation of `launch_standard_gamma_grad_kernel`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `launch_standard_gamma_grad_kernel`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 194-202
```cpp
 194: void launch_dirichlet_grad_kernel(TensorIteratorBase &iter) {
 195:   AT_DISPATCH_FLOATING_TYPES(iter.input_dtype(), "_dirichlet_grad_cuda", [&] {
 196:     using accscalar_t = at::acc_type<scalar_t, true>;
 197:     at::native::gpu_kernel(iter,
 198:       [] GPU_LAMBDA (scalar_t x_val, scalar_t alpha_val, scalar_t total_val) -> scalar_t {
 199:         return dirichlet_grad_one<scalar_t, accscalar_t>(x_val, alpha_val, total_val);
 200:       });
 201:   });
 202: }
```
- EN: This block defines or continues the implementation of `launch_dirichlet_grad_kernel`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `launch_dirichlet_grad_kernel`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 204-204
```cpp
 204: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TensorIterator` normalizes tensor shapes/strides and drives elementwise CUDA traversal. / `TensorIterator` 统一张量形状与步长，并驱动逐元素 CUDA 遍历。
- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `gpu_kernel` applies a functor over iterator elements on CUDA. / `gpu_kernel` 在 CUDA 上对迭代器元素应用函数对象。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/native/cuda/Distributions.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/cuda/CUDAApplyUtils.cuh>`
  - `<ATen/AccumulateType.h>`
  - `<ATen/cuda/CUDAGeneratorImpl.h>`
  - `<ATen/native/UnaryOps.h>`
  - `<ATen/native/cuda/DistributionTemplates.h>`
  - `<curand.h>`
  - `<curand_kernel.h>`
  - `<curand_philox4x32_x.h>`
  - `<utility>`
  - `<functional>`
- Runtime symbols / 运行时符号:
  - `launch_dirichlet_kernel`
  - `launch_poisson_cuda_kernel`
  - `launch_binomial_cuda_kernel`
  - `launch_gamma_kernel`
  - `launch_standard_gamma_grad_kernel`
  - `launch_dirichlet_grad_kernel`
  - `gpu_kernel`
  - `TensorIterator`
  - `TensorIteratorBase`
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `AT_DISPATCH_FLOATING_TYPES`
  - `at::cuda::philox::unpack`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
