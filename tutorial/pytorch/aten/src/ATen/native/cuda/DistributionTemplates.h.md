# DistributionTemplates.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/DistributionTemplates.h`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Declares or defines CUDA helpers/templates associated with `calc_execution_policy`, `distribution_nullary_kernel`, `distribution_binary_elementwise_kernel`, `distribution_binary_kernel`.
- 用途（中文）: 声明或定义与 `calc_execution_policy`, `distribution_nullary_kernel`, `distribution_binary_elementwise_kernel`, `distribution_binary_kernel` 相关的 CUDA 辅助函数/模板。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #pragma once
   2: 
   3: #include <ATen/AccumulateType.h>
   4: #include <ATen/Dispatch.h>
   5: #include <ATen/Dispatch_v2.h>
   6: #include <ATen/ExpandBase.h>
   7: #include <ATen/OpMathType.h>
   8: #include <ATen/native/TensorIterator.h>
   9: #include <ATen/native/cuda/Loops.cuh>
  10: #include <c10/util/Half.h>
  11: #include <ATen/cuda/CUDAApplyUtils.cuh>
  12: #include <ATen/cuda/CUDAContext.h>
  13: #include <ATen/cuda/detail/OffsetCalculator.cuh>
  14: #include <ATen/cuda/CUDAGraphsUtils.cuh>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/AccumulateType.h>`, `<ATen/Dispatch.h>`, `<ATen/Dispatch_v2.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/AccumulateType.h>`, `<ATen/Dispatch.h>`, `<ATen/Dispatch_v2.h>`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 15-26
```cpp
  15: #include <ATen/detail/FunctionTraits.h>
  16: #include <ATen/core/DistributionsHelper.h>
  17: 
  18: #include <curand.h>
  19: #include <curand_kernel.h>
  20: #include <curand_philox4x32_x.h>
  21: #include <cstdint>
  22: #include <limits>
  23: #include <utility>
  24: #include <mutex>
  25: #include <tuple>
  26: #include <type_traits>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/detail/FunctionTraits.h>`, `<ATen/core/DistributionsHelper.h>`, `<curand.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/detail/FunctionTraits.h>`, `<ATen/core/DistributionsHelper.h>`, `<curand.h>`。

### Lines 28-49
```cpp
  28: namespace at {
  29: namespace native {
  30: namespace {
  31: 
  32: // launch bounds used for kernels utilizing TensorIterator
  33: const uint32_t block_size_bound = 256;
  34: const uint32_t grid_size_bound = 4;
  35: // At the time of writing, there is no curand_* call that increments the offset by more than 4.
  36: // See: https://docs.nvidia.com/cuda/archive/11.8.0/curand/group__DEVICE.html
  37: const uint32_t max_generator_offsets_per_curand_call = 4;
  38: 
  39: // utility function that calculates proper philox_offset
  40: // for distributions utilizing TensorIterator. For distributions using
  41: // TensorIterator, we are using a grid-stride loop with each
  42: // thread yielding one element per thread. For the edge of the grid-stride
  43: // loop, if the tensor size is large, the unroll loop will kick in and the float4
  44: // from curand4 will start getting utilized (for common tensor sizes, we end up
  45: // using rand.x from each thread). The philox_offset calculation was changed to
  46: // (number of elements per thread * maximum generator increment per "curand_*" call), which makes
  47: // sure that philox offset increment is not less than the number of randoms used
  48: // in each thread.
  49: std::tuple<uint64_t, dim3, dim3> calc_execution_policy(const int64_t total_elements, const uint32_t unroll_factor) {
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `utilized`.
- CN: 该代码块定义或继续实现 `utilized`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 50-61
```cpp
  50:   const uint64_t numel = static_cast<uint64_t>(total_elements);
  51:   const uint32_t block_size = block_size_bound;
  52:   dim3 dim_block(block_size);
  53:   dim3 grid((numel + block_size - 1) / block_size);
  54:   uint32_t blocks_per_sm = at::cuda::getCurrentDeviceProperties()->maxThreadsPerMultiProcessor / block_size;
  55:   grid.x = std::min(
  56:       static_cast<uint32_t>(at::cuda::getCurrentDeviceProperties()->multiProcessorCount) * blocks_per_sm,
  57:       grid.x);
  58:   //number of times random will be generated per thread, to offset philox counter in thc random state
  59:   uint64_t counter_offset = ((numel - 1) / (block_size * grid.x * unroll_factor) + 1) * max_generator_offsets_per_curand_call;
  60:   return std::make_tuple(counter_offset, grid, dim_block);
  61: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 63-63
```cpp
  63: // grid stride loop kernel for distributions
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 64-85
```cpp
  64: template<typename accscalar_t, int unroll_factor, typename dist_t, typename transform_t>
  65: C10_LAUNCH_BOUNDS_2(block_size_bound, grid_size_bound)
  66: __global__ void distribution_elementwise_grid_stride_kernel(int64_t numel,
  67:                                                             PhiloxCudaState philox_args,
  68:                                                             const dist_t dist_func,
  69:                                                             const transform_t transform_func) {
  70:   auto [seed, offset] = at::cuda::philox::unpack(philox_args);
  71:   int64_t idx = ((int64_t) blockIdx.x) * blockDim.x + threadIdx.x;
  72:   curandStatePhilox4_32_10_t state;
  73:   curand_init(seed, idx, offset, &state);
  74: 
  75:   int64_t rounded_size = ((numel - 1)/(blockDim.x * gridDim.x * unroll_factor)+1) *
  76:       blockDim.x * gridDim.x * unroll_factor;
  77:   for(int64_t linear_index = idx; linear_index < rounded_size; linear_index += blockDim.x * gridDim.x * unroll_factor) {
  78:     auto rand = dist_func(&state);
  79:     #pragma unroll
  80:     for (int ii = 0; ii < unroll_factor; ii++) {
  81:       int64_t li = linear_index + blockDim.x * gridDim.x * ii;
  82:       if (li < numel) {
  83:         transform_func(li, static_cast<accscalar_t>((&rand.x)[ii]));
  84:       }
  85:     }
```
- EN: This block defines GPU kernel entry point(s) `distribution_elementwise_grid_stride_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `distribution_elementwise_grid_stride_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 86-88
```cpp
  86:     __syncthreads();
  87:   }
  88: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 90-105
```cpp
  90: /**
  91:  * distribution_nullary_kernel is analogous to gpu_kernel in
  92:  * ATen/native/cuda/Loops.cuh. Like gpu_kernel, it uses
  93:  * TensorIterator to launch a kernel. However, the differences are
  94:  *   - it launches a grid-stride loop based kernel. The kernel is not
  95:  *     generic like elementwise_kernel in Loops.cuh and is specialized
  96:  *     for the distribution kernels here.
  97:  *   - For big size tensors, we can launch multiple kernels recursively
  98:  *     (i.e. if (!iter.can_use_32bit_indexing())) and hence, the philox
  99:  *     offset calculation is done in this function.
 100:  *
 101:  * FIXME: Can we specialize elementwise_kernel and launch_kernel in Loops.cuh
 102:  * to have grid-stride loop kernel and then use that to launch our distribution
 103:  * kernels? Note that we need a grid-stride loop kernel because, we found by testing
 104:  * that it achieves peak effective bandwidth.
 105:  */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 106-127
```cpp
 106: template<typename scalar_t,
 107:          typename accscalar_t,
 108:          typename dist_func_return_t,
 109:          typename RNG,
 110:          typename dist_t,
 111:          typename transform_t>
 112: void distribution_nullary_kernel(at::TensorIteratorBase& iter,
 113:                                  RNG gen,
 114:                                  const dist_t& dist_func,
 115:                                  const transform_t transform_func) {
 116:   const int unroll_factor = sizeof(dist_func_return_t) / sizeof(accscalar_t);
 117:   TORCH_CHECK(unroll_factor >= 1, "unroll_factor must be >= 1.");
 118:   int64_t numel = iter.numel();
 119:   if (numel == 0) {
 120:     return;
 121:   }
 122: 
 123:   auto [counter_offset, grid, block] = calc_execution_policy(numel, unroll_factor);
 124:   PhiloxCudaState rng_engine_inputs;
 125:   {
 126:     // See Note [Acquire lock when using random generators]
 127:     std::lock_guard<std::mutex> lock(gen->mutex_);
```
- EN: This block defines or continues the implementation of `distribution_nullary_kernel`.
- CN: 该代码块定义或继续实现 `distribution_nullary_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 128-129
```cpp
 128:     rng_engine_inputs = gen->philox_cuda_state(counter_offset);
 129:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 131-137
```cpp
 131:   if (!iter.can_use_32bit_indexing()) {
 132:     for (auto& sub_iter : iter.with_32bit_indexing()) {
 133:       distribution_nullary_kernel<scalar_t, accscalar_t, dist_func_return_t>(sub_iter,
 134:         gen, dist_func, transform_func);
 135:     }
 136:     return;
 137:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 139-139
```cpp
 139:   char* out_data = (char*)iter.data_ptr(0);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 141-162
```cpp
 141:   auto stream = at::cuda::getCurrentCUDAStream();
 142:   if (iter.is_trivial_1d()) {
 143:     auto strides = iter.get_inner_strides();
 144:     int stride0 = strides[0];
 145:     distribution_elementwise_grid_stride_kernel<accscalar_t, unroll_factor><<<grid, block, 0, stream>>>(
 146:       numel,
 147:       rng_engine_inputs,
 148:       dist_func,
 149:       [=]__device__(int idx, accscalar_t rand) {
 150:         scalar_t* out = (scalar_t*)&out_data[stride0 * idx];
 151:         *out = transform_func(rand);
 152:       }
 153:     );
 154:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 155:   } else {
 156:     auto offset_calc = make_offset_calculator<1>(iter);
 157:     distribution_elementwise_grid_stride_kernel<accscalar_t, unroll_factor><<<grid, block, 0, stream>>>(
 158:       numel,
 159:       rng_engine_inputs,
 160:       dist_func,
 161:       [=]__device__(int idx, accscalar_t rand) {
 162:         auto offsets = offset_calc.get(idx);
```
- EN: This block defines or continues the implementation of `__device__`.
- CN: 该代码块定义或继续实现 `__device__`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 163-169
```cpp
 163:         scalar_t* out = (scalar_t*)&out_data[offsets[0]];
 164:         *out = transform_func(rand);
 165:       }
 166:     );
 167:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 168:   }
 169: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 171-171
```cpp
 171: // Binary kernel
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 172-193
```cpp
 172: template <typename func_t, typename inp_offset_calc_t, typename out_offset_calc_t>
 173: __global__ void distribution_binary_elementwise_kernel(
 174:     int numel,
 175:     func_t f,
 176:     PhiloxCudaState philox_args,
 177:     typename function_traits<func_t>::result_type *output_data,
 178:     const typename function_traits<func_t>::template arg<1>::type *input_data_1,
 179:     const typename function_traits<func_t>::template arg<2>::type *input_data_2,
 180:     inp_offset_calc_t inp_calc,
 181:     out_offset_calc_t out_calc) {
 182:   auto seeds = at::cuda::philox::unpack(philox_args);
 183: 
 184:   using input_t_1 = typename function_traits<func_t>::template arg<1>::type;
 185:   using input_t_2 = typename function_traits<func_t>::template arg<2>::type;
 186: 
 187:   input_t_1 inputs_1[thread_work_size()];
 188:   input_t_2 inputs_2[thread_work_size()];
 189: 
 190:   int base_index = block_work_size() * blockIdx.x;
 191:   int remaining = std::min<int>(numel - base_index, block_work_size());
 192: 
 193:   curandStatePhilox4_32_10_t state;
```
- EN: This block defines GPU kernel entry point(s) `distribution_binary_elementwise_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `distribution_binary_elementwise_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 194-197
```cpp
 194:   curand_init(std::get<0>(seeds),
 195:               blockIdx.x * blockDim.x + threadIdx.x,
 196:               std::get<1>(seeds),
 197:               &state);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 199-199
```cpp
 199:   // load data into registers
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 200-212
```cpp
 200:   int thread_idx = threadIdx.x;
 201:   #pragma unroll
 202:   for (int i = 0; i < thread_work_size(); i++) {
 203:     if (thread_idx >= remaining) {
 204:       break;
 205:     }
 206:     int input_idx = thread_idx + base_index;
 207:     auto offsets = inp_calc.get(input_idx);
 208:     inputs_1[i] = input_data_1[offsets[0]];
 209:     inputs_2[i] = input_data_2[offsets[1]];
 210: 
 211:     thread_idx += num_threads();
 212:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 214-214
```cpp
 214:   // compute and store
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 215-226
```cpp
 215:   thread_idx = threadIdx.x;
 216:   #pragma unroll
 217:   for (int i = 0; i < thread_work_size(); i++) {
 218:     if (thread_idx >= remaining) {
 219:       break;
 220:     }
 221:     int input_idx = thread_idx + base_index;
 222:     auto offsets = out_calc.get(input_idx);
 223:     output_data[offsets[0]] = f(state, inputs_1[i], inputs_2[i]);
 224:     thread_idx += num_threads();
 225:   }
 226: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 228-249
```cpp
 228: template <typename func_t>
 229: void distribution_binary_kernel(TensorIteratorBase &iter, PhiloxCudaState philox_args, const func_t &f) {
 230:   static_assert(std::is_same_v<typename function_traits<func_t>::template arg<0>::type, curandStatePhilox4_32_10_t&>, "the first argument of functor must be curandStatePhilox4_32_10_t");
 231:   using input_t_1 = typename function_traits<func_t>::template arg<1>::type;
 232:   using input_t_2 = typename function_traits<func_t>::template arg<2>::type;
 233:   using output_t = typename function_traits<func_t>::result_type;
 234: 
 235:   if (!iter.can_use_32bit_indexing()) {
 236:     for (auto& sub_iter : iter.with_32bit_indexing()) {
 237:       distribution_binary_kernel(sub_iter, philox_args, f);
 238:     }
 239:     return;
 240:   }
 241: 
 242:   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(iter.can_use_32bit_indexing());
 243: 
 244:   int64_t numel = iter.numel();
 245:   if (numel == 0) {
 246:     return;
 247:   }
 248: 
 249:   output_t *output_data = static_cast<output_t *>(iter.data_ptr(0));
```
- EN: This block defines or continues the implementation of `distribution_binary_kernel`.
- CN: 该代码块定义或继续实现 `distribution_binary_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 250-251
```cpp
 250:   const input_t_1 *input_data_1 = static_cast<const input_t_1 *>(iter.data_ptr(1));
 251:   const input_t_2 *input_data_2 = static_cast<const input_t_2 *>(iter.data_ptr(2));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 253-254
```cpp
 253:   int64_t grid = (numel + block_work_size() - 1) / block_work_size();
 254:   auto stream = at::cuda::getCurrentCUDAStream();
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 256-267
```cpp
 256:   if (iter.is_contiguous()) {
 257:     distribution_binary_elementwise_kernel<<<grid,num_threads(), 0, stream>>>(
 258:         numel, f, philox_args, output_data, input_data_1, input_data_2,
 259:         TrivialOffsetCalculator<2>(), TrivialOffsetCalculator<1>());
 260:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 261:   } else {
 262:     distribution_binary_elementwise_kernel<<<grid, num_threads(), 0, stream>>>(
 263:         numel, f, philox_args, output_data, input_data_1, input_data_2,
 264:         make_input_offset_calculator<2>(iter), make_output_offset_calculator(iter));
 265:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 266:   }
 267: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 269-270
```cpp
 269: } // namespace
 270: }} // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 273-294
```cpp
 273: namespace at {
 274: namespace native {
 275: namespace templates {
 276: namespace cuda {
 277: 
 278: // ==================================================== Random ========================================================
 279: 
 280: template<typename RNG>
 281: void random_from_to_kernel(TensorIteratorBase& iter, uint64_t range, int64_t base, RNG gen) {
 282: #ifdef FBCODE_CAFFE2
 283:   AT_DISPATCH_V2(iter.dtype(), "random_from_to_kernel_cuda", AT_WRAP([&] {
 284:     if ((
 285:       std::is_same_v<scalar_t, int64_t> ||
 286:       std::is_same_v<scalar_t, double> ||
 287:       std::is_same_v<scalar_t, float> ||
 288:       std::is_same_v<scalar_t, at::BFloat16>) && range >= 1ULL << 32)
 289:     {
 290:       // define lambda to mod with range and add base
 291:       auto random_func = [range, base] __device__ (uint64_t rand) {
 292:         return transformation::uniform_int_from_to<scalar_t>(rand, range, base);
 293:       };
 294:       distribution_nullary_kernel<scalar_t, uint64_t, ulonglong2>(iter,
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `random_from_to_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `random_from_to_kernel`, `__device__`。

### Lines 295-308
```cpp
 295:         gen,
 296:         [] __device__ (curandStatePhilox4_32_10_t* state) -> ulonglong2 {
 297:           ulonglong2 ret;
 298:           uint4 rand_val = curand4(state);
 299:           ret.x = (static_cast<uint64_t>(rand_val.x) << 32) | rand_val.y;
 300:           ret.y = (static_cast<uint64_t>(rand_val.z) << 32) | rand_val.w;
 301:           return ret;
 302:         },
 303:         random_func);
 304:     } else {
 305:       auto random_func = [range, base] __device__ (uint32_t rand) {
 306:         return transformation::uniform_int_from_to<scalar_t>(rand, range, base);
 307:       };
 308:       distribution_nullary_kernel<scalar_t, uint32_t, uint4>(iter,
```
- EN: This block defines or continues the implementation of `__device__`.
- CN: 该代码块定义或继续实现 `__device__`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 309-323
```cpp
 309:         gen,
 310:         [] __device__ (curandStatePhilox4_32_10_t* state) -> uint4 {
 311:           return curand4(state);
 312:         },
 313:         random_func);
 314:     }
 315:    }), AT_EXPAND(AT_ALL_TYPES), kBool, kHalf, kBFloat16, AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES));
 316: #else
 317:   AT_DISPATCH_V2(iter.dtype(), "random_from_to_kernel_cuda", AT_WRAP([&] {
 318:     if (range >= 1ULL << 28) // allow approx 5% skew in uniform int generation using %
 319:     {
 320:       // define lambda to mod with range and add base
 321:       auto random_func = [range, base] __device__ (uint64_t rand) {
 322:         return transformation::uniform_int_from_to<scalar_t>(rand, range, base);
 323:       };
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `__device__`.
- CN: 该代码块定义或继续实现 `__device__`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 324-337
```cpp
 324:       distribution_nullary_kernel<scalar_t, uint64_t, ulonglong2>(iter,
 325:         gen,
 326:         [] __device__ (curandStatePhilox4_32_10_t* state) -> ulonglong2 {
 327:           ulonglong2 ret;
 328:           uint4 rand_val = curand4(state);
 329:           ret.x = (static_cast<uint64_t>(rand_val.x) << 32) | rand_val.y;
 330:           ret.y = (static_cast<uint64_t>(rand_val.z) << 32) | rand_val.w;
 331:           return ret;
 332:         },
 333:         random_func);
 334:     } else {
 335:       auto random_func = [range, base] __device__ (uint32_t rand) {
 336:         return transformation::uniform_int_from_to<scalar_t>(rand, range, base);
 337:       };
```
- EN: This block defines or continues the implementation of `__device__`.
- CN: 该代码块定义或继续实现 `__device__`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 338-347
```cpp
 338:       distribution_nullary_kernel<scalar_t, uint32_t, uint4>(iter,
 339:         gen,
 340:         [] __device__ (curandStatePhilox4_32_10_t* state) -> uint4 {
 341:           return curand4(state);
 342:         },
 343:         random_func);
 344:     }
 345:    }), AT_EXPAND(AT_ALL_TYPES), kBool, kHalf, kBFloat16, AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES));
 346: #endif
 347: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `__device__`.
- CN: 该代码块定义或继续实现 `__device__`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 349-351
```cpp
 349: // This is the special kernel to handle single specific case:
 350: // from(inclusive) = std::numeric_limits<int64_t>::lowest()
 351: // to(exclusive) = None (= std::numeric_limits<int64_t>::max() + 1)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 352-373
```cpp
 352: template<typename RNG>
 353: void random_full_64_bits_range_kernel(TensorIteratorBase& iter, RNG gen) {
 354:   AT_DISPATCH_ALL_TYPES_AND(at::ScalarType::BFloat16, iter.dtype(), "random_full_64_bits_range_kernel_cuda", [&] {
 355:     if (std::is_same_v<scalar_t, int64_t> ||
 356:         std::is_same_v<scalar_t, double> ||
 357:         std::is_same_v<scalar_t, float> ||
 358:         std::is_same_v<scalar_t, at::BFloat16>) {
 359:       auto random_func = [] __device__ (uint64_t rand) {
 360:         return transformation::uniform_int_full_range<scalar_t>(rand);
 361:       };
 362:       distribution_nullary_kernel<scalar_t, uint64_t, ulonglong2>(iter,
 363:         gen,
 364:         [] __device__ (curandStatePhilox4_32_10_t* state) -> ulonglong2 {
 365:           ulonglong2 ret;
 366:           uint4 rand_val = curand4(state);
 367:           ret.x = (static_cast<uint64_t>(rand_val.x) << 32) | rand_val.y;
 368:           ret.y = (static_cast<uint64_t>(rand_val.z) << 32) | rand_val.w;
 369:           return ret;
 370:         },
 371:         random_func);
 372:     } else {
 373:       TORCH_CHECK(false, "random_full_64_bits_range_kernel_cuda handles only int64, double, float and bfloat16");
```
- EN: This block defines or continues the implementation of `random_full_64_bits_range_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `random_full_64_bits_range_kernel`, `__device__`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 374-376
```cpp
 374:     }
 375:   });
 376: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 378-386
```cpp
 378: template<typename RNG>
 379: struct RandomFromToKernel {
 380:   void operator()(TensorIteratorBase& iter, uint64_t range, int64_t base, std::optional<Generator> gen) {
 381:     random_from_to_kernel(iter, range, base, check_generator<RNG>(gen));
 382:   }
 383:   void operator()(TensorIteratorBase& iter, std::optional<Generator> gen) {
 384:     random_full_64_bits_range_kernel(iter, check_generator<RNG>(gen));
 385:   }
 386: };
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 388-409
```cpp
 388: template<typename RNG>
 389: void random_kernel(TensorIteratorBase& iter, RNG gen) {
 390:   AT_DISPATCH_ALL_TYPES_AND3(at::ScalarType::Half, at::ScalarType::BFloat16, at::ScalarType::Bool, iter.dtype(), "random_kernel_cuda", [&] {
 391:     if (std::is_same_v<scalar_t, double> || std::is_same_v<scalar_t, int64_t>) {
 392:       auto random_func = [] __device__ (uint64_t rand) {
 393:         return transformation::uniform_int<scalar_t>(rand);
 394:       };
 395:       distribution_nullary_kernel<scalar_t, uint64_t, ulonglong2>(iter, gen,
 396:         [] __device__ (curandStatePhilox4_32_10_t* state) -> ulonglong2 {
 397:           ulonglong2 ret;
 398:           uint4 rand_val = curand4(state);
 399:           ret.x = (static_cast<uint64_t>(rand_val.x) << 32) | rand_val.y;
 400:           ret.y = (static_cast<uint64_t>(rand_val.z) << 32) | rand_val.w;
 401:           return ret;
 402:         },
 403:         random_func);
 404:     } else {
 405:       auto random_func = [] __device__ (uint32_t rand) {
 406:         return transformation::uniform_int<scalar_t>(rand);
 407:       };
 408:       distribution_nullary_kernel<scalar_t, uint32_t, uint4>(iter,
 409:         gen,
```
- EN: This block defines or continues the implementation of `random_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `random_kernel`, `__device__`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 410-416
```cpp
 410:         [] __device__ (curandStatePhilox4_32_10_t* state) -> uint4 {
 411:           return curand4(state);
 412:         },
 413:         random_func);
 414:     }
 415:   });
 416: }
```
- EN: This block defines or continues the implementation of `__device__`.
- CN: 该代码块定义或继续实现 `__device__`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 418-423
```cpp
 418: template<typename RNG>
 419: struct RandomKernel {
 420:   void operator()(TensorIteratorBase& iter, RNG gen) {
 421:     random_kernel(iter, gen);
 422:   }
 423: };
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 425-425
```cpp
 425: // ====================================================================================================================
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 427-440
```cpp
 427: template<typename scalar_t, typename accscalar_t, typename RNG, typename transform_t>
 428: void uniform_and_transform(TensorIteratorBase& iter, RNG gen, transform_t transform) {
 429:   if (std::is_same_v<scalar_t, double>) {
 430:     distribution_nullary_kernel<scalar_t, accscalar_t, double2>(iter,
 431:       gen,
 432:       [] __device__ (curandStatePhilox4_32_10_t* state) -> double2 { return curand_uniform2_double(state); },
 433:       transform);
 434:   } else {
 435:     distribution_nullary_kernel<scalar_t, accscalar_t, float4>(iter,
 436:       gen,
 437:       [] __device__ (curandStatePhilox4_32_10_t* state) -> float4 { return curand_uniform4(state); },
 438:       transform);
 439:   }
 440: }
```
- EN: This block defines or continues the implementation of `uniform_and_transform`, `__device__`.
- CN: 该代码块定义或继续实现 `uniform_and_transform`, `__device__`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 442-455
```cpp
 442: template<typename scalar_t, typename accscalar_t, typename RNG, typename transform_t>
 443: void normal_and_transform(TensorIteratorBase& iter, RNG gen, transform_t transform) {
 444:   if (std::is_same_v<scalar_t, double>) {
 445:     distribution_nullary_kernel<scalar_t, accscalar_t, double2>(iter,
 446:       gen,
 447:       [] __device__ (curandStatePhilox4_32_10_t* state) -> double2 { return curand_normal2_double(state); },
 448:       transform);
 449:   } else {
 450:     distribution_nullary_kernel<scalar_t, accscalar_t, float4>(iter,
 451:       gen,
 452:       [] __device__ (curandStatePhilox4_32_10_t* state) -> float4 { return curand_normal4(state); },
 453:       transform);
 454:   }
 455: }
```
- EN: This block defines or continues the implementation of `normal_and_transform`, `__device__`.
- CN: 该代码块定义或继续实现 `normal_and_transform`, `__device__`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 457-457
```cpp
 457: // ==================================================== Normal ========================================================
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 459-472
```cpp
 459: template<typename RNG>
 460: void normal_kernel(const TensorBase &self, double mean_, double std_, RNG gen) {
 461:   auto iter = TensorIterator::borrowing_nullary_op(self);
 462:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, iter.dtype(), "normal_kernel_cuda", [&] {
 463:     using accscalar_t = at::acc_type<scalar_t, true>;
 464:     auto mean = static_cast<accscalar_t>(mean_);
 465:     auto std = static_cast<accscalar_t>(std_);
 466:     // define lambda to multiply std and add mean
 467:     auto normal_func = [mean, std] __device__ (accscalar_t rand) {
 468:       return static_cast<scalar_t>(transformation::normal<accscalar_t>(rand, mean, std));
 469:     };
 470:     normal_and_transform<scalar_t, accscalar_t>(iter, gen, normal_func);
 471:    });
 472: }
```
- EN: This block defines or continues the implementation of `normal_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `normal_kernel`, `__device__`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 474-479
```cpp
 474: template<typename RNG>
 475: struct NormalKernel {
 476:   void operator()(const TensorBase &self, double mean, double std, std::optional<Generator> gen) {
 477:     normal_kernel(self, mean, std, check_generator<RNG>(gen));
 478:   }
 479: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 481-481
```cpp
 481: // ==================================================== Uniform ========================================================
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 483-504
```cpp
 483: template<typename RNG>
 484: void uniform_kernel(TensorIteratorBase& iter, double from_, double to_, RNG gen) {
 485:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, iter.dtype(), "uniform_kernel_cuda", [&] {
 486:     auto from = static_cast<scalar_t>(from_);
 487:     auto to = static_cast<scalar_t>(to_);
 488:     using opmath_t = at::opmath_type<scalar_t>;
 489:     auto range = static_cast<opmath_t>(to-from);
 490:     // define lambda to reverse bounds, multiply 'range' and add 'from_'
 491:     auto uniform_func = [range, from, to] __device__ (opmath_t rand) {
 492:       // Compute output value before reversing the bounds
 493:       // BEFORE TOUCHING THIS CODE READ: https://github.com/pytorch/pytorch/issues/96947
 494:       auto value = static_cast<scalar_t>(rand * range + from);
 495:       // reverse the bounds of curand4 from (0, 1] to [0, 1)
 496:       // Note that this method is from legacy THCTensorRandom and is likely to give
 497:       // you more 0-s, since, the probability of getting 1-s is higher than 0-s and
 498:       // by reversing the bounds, we are flipping the probabilities of 1-s and 0-s.
 499:       // BEFORE TOUCHING THIS CODE READ: https://github.com/pytorch/pytorch/issues/16706
 500:       auto reverse_bound_value = value == to ? from : value;
 501:       return reverse_bound_value;
 502:     };
 503:     uniform_and_transform<scalar_t, opmath_t>(iter, gen, uniform_func);
 504:    });
```
- EN: This block defines or continues the implementation of `uniform_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `uniform_kernel`, `__device__`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 505-505
```cpp
 505: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 507-512
```cpp
 507: template<typename RNG>
 508: struct UniformKernel {
 509:   void operator()(TensorIteratorBase& iter, double from, double to, std::optional<Generator> gen) {
 510:     uniform_kernel(iter, from, to, check_generator<RNG>(gen));
 511:   }
 512: };
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 514-514
```cpp
 514: // ================================================== LogNormal =======================================================
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 516-528
```cpp
 516: template<typename RNG>
 517: void log_normal_kernel(TensorIteratorBase& iter, double mean_, double std_, RNG gen) {
 518:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, iter.dtype(), "log_normal_cuda", [&] {
 519:     using accscalar_t = at::acc_type<scalar_t, true>;
 520:     auto mean = static_cast<accscalar_t>(mean_);
 521:     auto std = static_cast<accscalar_t>(std_);
 522:     // define lambda for log_normal transformation
 523:     auto log_normal_func = [mean, std] __device__ (accscalar_t rand) {
 524:       return static_cast<scalar_t>(transformation::log_normal<accscalar_t>(transformation::normal<accscalar_t>(rand, mean, std)));
 525:     };
 526:     normal_and_transform<scalar_t, accscalar_t>(iter, gen, log_normal_func);
 527:    });
 528: }
```
- EN: This block defines or continues the implementation of `log_normal_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `log_normal_kernel`, `__device__`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 530-535
```cpp
 530: template<typename RNG>
 531: struct LogNormalKernel {
 532:   void operator()(TensorIteratorBase& iter, double mean, double std, std::optional<Generator> gen) {
 533:     log_normal_kernel(iter, mean, std, check_generator<RNG>(gen));
 534:   }
 535: };
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 537-537
```cpp
 537: // =================================================== Geometric ======================================================
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 539-549
```cpp
 539: template<typename RNG>
 540: void geometric_kernel(TensorIteratorBase& iter, double p, RNG gen) {
 541:   AT_DISPATCH_ALL_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, iter.dtype(), "geometric_cuda", [&] {
 542:     using accscalar_t = at::DiscreteDistributionType<scalar_t>::type;
 543:     // define lambda for geometric transformation
 544:     auto geometric_func = [p] __device__ (accscalar_t rand) {
 545:       return static_cast<scalar_t>(transformation::geometric<accscalar_t>(rand, p));
 546:     };
 547:     uniform_and_transform<scalar_t, accscalar_t>(iter, gen, geometric_func);
 548:   });
 549: }
```
- EN: This block defines or continues the implementation of `geometric_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `geometric_kernel`, `__device__`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 551-556
```cpp
 551: template<typename RNG>
 552: struct GeometricKernel {
 553:   void operator()(TensorIteratorBase& iter, double p, std::optional<Generator> gen) {
 554:     geometric_kernel(iter, p, check_generator<RNG>(gen));
 555:   }
 556: };
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 558-558
```cpp
 558: // ================================================== Exponential =====================================================
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 560-572
```cpp
 560: template<typename RNG>
 561: void exponential_kernel(TensorIteratorBase& iter, double lambda_, RNG gen) {
 562:   TORCH_CHECK(isFloatingType(iter.dtype()), "Exponential distribution is a continuous probability distribution. dtype must be a floating point but you specified ", iter.dtype());
 563:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, iter.dtype(), "exponential_cuda", [&] {
 564:     using accscalar_t = at::acc_type<scalar_t, true>;
 565:     auto lambda = static_cast<accscalar_t>(lambda_);
 566:     // define lambda for exponential transformation
 567:     auto exponential_func = [lambda] __device__ (accscalar_t rand) {
 568:       return static_cast<scalar_t>(transformation::exponential<accscalar_t>(rand, lambda));
 569:     };
 570:     uniform_and_transform<scalar_t, accscalar_t>(iter, gen, exponential_func);
 571:    });
 572: }
```
- EN: This block defines or continues the implementation of `exponential_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `exponential_kernel`, `__device__`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 574-579
```cpp
 574: template<typename RNG>
 575: struct ExponentialKernel {
 576:   void operator()(TensorIteratorBase& iter, double lambda, std::optional<Generator> gen) {
 577:     exponential_kernel(iter, lambda, check_generator<RNG>(gen));
 578:   }
 579: };
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 581-581
```cpp
 581: // ==================================================== Cauchy ========================================================
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 583-595
```cpp
 583: template<typename RNG>
 584: void cauchy_kernel(TensorIteratorBase& iter, double median_, double sigma_, RNG gen) {
 585:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, iter.dtype(), "cauchy_cuda", [&] {
 586:     using accscalar_t = at::acc_type<scalar_t, true>;
 587:     auto median = static_cast<accscalar_t>(median_);
 588:     auto sigma = static_cast<accscalar_t>(sigma_);
 589:     // define lambda for cauchy transformation
 590:     auto cauchy_func = [median, sigma] __device__ (accscalar_t rand) {
 591:       return static_cast<scalar_t>(transformation::cauchy<accscalar_t>(rand, median, sigma));
 592:     };
 593:     uniform_and_transform<scalar_t, accscalar_t>(iter, gen, cauchy_func);
 594:    });
 595: }
```
- EN: This block defines or continues the implementation of `cauchy_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `cauchy_kernel`, `__device__`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 597-602
```cpp
 597: template<typename RNG>
 598: struct CauchyKernel {
 599:   void operator()(TensorIteratorBase& iter, double median, double sigma, std::optional<Generator> gen) {
 600:     cauchy_kernel(iter, median, sigma, check_generator<RNG>(gen));
 601:   }
 602: };
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 604-604
```cpp
 604: // ==================================================== Bernoulli =====================================================
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 606-627
```cpp
 606: template<typename scalar_t, typename prob_t>
 607: void bernoulli_tensor_cuda_kernel(
 608:     const TensorBase &ret, const at::TensorBase &p,
 609:     PhiloxCudaState philox_args) {
 610:   auto functor = [philox_args] __device__(
 611:           int n, scalar_t& v1, scalar_t& v2, scalar_t& v3, scalar_t& v4,
 612:           const prob_t& p1, const prob_t& p2, const prob_t& p3, const prob_t& p4) {
 613:         auto seeds = at::cuda::philox::unpack(philox_args);
 614:         curandStatePhilox4_32_10_t state;
 615:         curand_init(std::get<0>(seeds),
 616:                     blockIdx.x * blockDim.x + threadIdx.x,
 617:                     std::get<1>(seeds),
 618:                     &state);
 619: 
 620:         // See Note [Register spilling in curand call for CUDA < 10]
 621:         float4 rand = curand_uniform4(&state);
 622:         switch (n) {
 623:           case 4: {
 624:             CUDA_KERNEL_ASSERT(0 <= p4 && p4 <= 1);
 625:             v4 = static_cast<scalar_t>(rand.w <= p4);
 626:             [[fallthrough]];
 627:           }
```
- EN: This block defines or continues the implementation of `bernoulli_tensor_cuda_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `bernoulli_tensor_cuda_kernel`, `__device__`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 628-641
```cpp
 628:           case 3: {
 629:             CUDA_KERNEL_ASSERT(0 <= p3 && p3 <= 1);
 630:             v3 = static_cast<scalar_t>(rand.z <= p3);
 631:             [[fallthrough]];
 632:           }
 633:           case 2: {
 634:             CUDA_KERNEL_ASSERT(0 <= p2 && p2 <= 1);
 635:             v2 = static_cast<scalar_t>(rand.y <= p2);
 636:             [[fallthrough]];
 637:           }
 638:           case 1: {
 639:             CUDA_KERNEL_ASSERT(0 <= p1 && p1 <= 1);
 640:             v1 = static_cast<scalar_t>(rand.x <= p1);
 641:           }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 642-649
```cpp
 642:         }
 643:       };
 644:   // The template argument `4` below indicates that we want to operate on four
 645:   // element at each time. See NOTE [ CUDA_tensor_applyN helpers ] for details.
 646:   at::cuda::CUDA_tensor_apply2<scalar_t, const prob_t, 4, decltype(functor),
 647:                                /*max_threads_per_block=*/512,
 648:                                /*min_blocks_per_sm==*/2>(ret, p, functor);
 649: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 651-672
```cpp
 651: template<typename RNG>
 652: void bernoulli_kernel(const TensorBase &self, const TensorBase &p_, RNG gen) {
 653:   PhiloxCudaState rng_engine_inputs;
 654:   {
 655:     // See Note [Acquire lock when using random generators]
 656:     std::lock_guard<std::mutex> lock(gen->mutex_);
 657:     rng_engine_inputs = gen->philox_cuda_state(10);
 658:   }
 659:   TORCH_CHECK(at::isFloatingType(p_.scalar_type()), "expected probabilities tensor to have floating type, got ", p_.scalar_type());
 660:   // cast probabilities tensor to double for double `self` tensor, and to `float` for everything else
 661:   const auto p_type = self.dtype() == at::kDouble ? at::kDouble : at::kFloat;
 662:   auto p_cuda = p_.to(TensorOptions().device(self.device()).dtype(p_type));
 663:   auto p = expand_inplace(self, p_cuda);
 664:   AT_DISPATCH_ALL_TYPES_AND3(
 665:     at::ScalarType::Half, at::ScalarType::BFloat16, at::ScalarType::Bool, self.scalar_type(), "bernoulli_tensor_cuda_self_", [&] {
 666:       if (std::is_same_v<scalar_t, double>) {
 667:         return bernoulli_tensor_cuda_kernel<double, double>(self, *p, rng_engine_inputs);
 668:       } else {
 669:         return bernoulli_tensor_cuda_kernel<scalar_t, float>(self, *p, rng_engine_inputs);
 670:       }
 671:    });
 672: }
```
- EN: This block defines or continues the implementation of `bernoulli_kernel`.
- CN: 该代码块定义或继续实现 `bernoulli_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 674-685
```cpp
 674: template<typename RNG>
 675: void bernoulli_kernel(TensorIteratorBase& iter, double p, RNG gen) {
 676:   AT_DISPATCH_ALL_TYPES_AND3(
 677:     at::ScalarType::Half, at::ScalarType::BFloat16, at::ScalarType::Bool, iter.dtype(), "bernoulli_scalar_cuda_", [&] {
 678:       using accscalar_t = at::DiscreteDistributionType<scalar_t>::type;
 679:       // define lambda for bernoulli transformation
 680:       auto bernoulli_func = [p] __device__ (accscalar_t rand) {
 681:         return static_cast<scalar_t>(transformation::bernoulli<accscalar_t>(rand, p));
 682:       };
 683:       uniform_and_transform<scalar_t, accscalar_t>(iter, gen, bernoulli_func);
 684:    });
 685: }
```
- EN: This block defines or continues the implementation of `bernoulli_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `bernoulli_kernel`, `__device__`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 687-695
```cpp
 687: template<typename RNG>
 688: struct BernoulliKernel {
 689:   void operator()(TensorIteratorBase& iter, double p, std::optional<Generator> gen) {
 690:     bernoulli_kernel(iter, p, check_generator<RNG>(gen));
 691:   }
 692:   void operator()(const TensorBase &self, const TensorBase &p_, std::optional<Generator> gen) {
 693:     bernoulli_kernel(self, p_, check_generator<RNG>(gen));
 694:   }
 695: };
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 697-697
```cpp
 697: }}}}
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

## Key Concepts / 关键概念

- `TensorIterator` normalizes tensor shapes/strides and drives elementwise CUDA traversal. / `TensorIterator` 统一张量形状与步长，并驱动逐元素 CUDA 遍历。
- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `gpu_kernel` applies a functor over iterator elements on CUDA. / `gpu_kernel` 在 CUDA 上对迭代器元素应用函数对象。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/AccumulateType.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/Dispatch_v2.h>`
  - `<ATen/ExpandBase.h>`
  - `<ATen/OpMathType.h>`
  - `<ATen/native/TensorIterator.h>`
  - `<ATen/native/cuda/Loops.cuh>`
  - `<c10/util/Half.h>`
  - `<ATen/cuda/CUDAApplyUtils.cuh>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/detail/OffsetCalculator.cuh>`
  - `<ATen/cuda/CUDAGraphsUtils.cuh>`
- Runtime symbols / 运行时符号:
  - `launch_kernel`
  - `gpu_kernel`
  - `TensorIterator`
  - `TensorIteratorBase`
  - `AT_DISPATCH_V2`
  - `AT_DISPATCH_ALL_TYPES_AND`
  - `AT_DISPATCH_ALL_TYPES_AND3`
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `AT_DISPATCH_ALL_TYPES_AND2`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::philox::unpack`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
