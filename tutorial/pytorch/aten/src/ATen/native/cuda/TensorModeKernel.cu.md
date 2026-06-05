# TensorModeKernel.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/TensorModeKernel.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `calculate_mode`, `apply_mode`, `handle_fused_mode`, `fused_mode`.
- 用途（中文）: 实现与 `calculate_mode`, `apply_mode`, `handle_fused_mode`, `fused_mode` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
```cpp
   1: #define TORCH_ASSERT_NO_OPERATORS
   2: #include <ATen/native/cuda/TensorModeKernel.cuh>
   3: #include <ATen/native/cuda/TensorModeKernel.h>
   4: #include <ATen/Dispatch.h>
   5: #include <ATen/native/NonEmptyUtils.h>
   6: #include <ATen/cuda/detail/IndexUtils.cuh>
   7: #include <ATen/cuda/ThrustAllocator.h>
   8: #include <c10/core/DeviceArray.h>
   9: 
  10: #include <thrust/count.h>
  11: #include <thrust/device_ptr.h>
  12: #include <thrust/device_vector.h>
  13: #include <thrust/execution_policy.h>
  14: #include <thrust/extrema.h>
  15: #include <thrust/find.h>
  16: #include <thrust/inner_product.h>
  17: #include <thrust/iterator/constant_iterator.h>
  18: #include <thrust/sequence.h>
  19: #include <thrust/sort.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/TensorModeKernel.cuh>`, `<ATen/native/cuda/TensorModeKernel.h>`, `<ATen/Dispatch.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/TensorModeKernel.cuh>`, `<ATen/native/cuda/TensorModeKernel.h>`, `<ATen/Dispatch.h>`。

### Lines 21-42
```cpp
  21: namespace at::native {
  22: 
  23: template <typename scalar_t>
  24: struct ModeImpl {
  25:   std::tuple<scalar_t, int64_t> operator()(
  26:       scalar_t *iter_begin,
  27:       scalar_t *iter_end) {
  28:     at::cuda::ThrustAllocator thrust_allocator;
  29:     auto stream = at::cuda::getCurrentCUDAStream();
  30:     auto policy = thrust::cuda::par(thrust_allocator).on(stream);
  31: 
  32:     const auto n_element = iter_end - iter_begin;
  33:     auto cuda_allocator = at::cuda::getCUDADeviceAllocator();
  34:     auto sort_buffer = c10::DeviceArray<int64_t>(*cuda_allocator, n_element);
  35:     auto sort_buffer_ptr = thrust::device_pointer_cast(sort_buffer.get());
  36:     auto count_from_zero_iter = thrust::make_counting_iterator(int64_t{0});
  37:     thrust::copy_n(policy, count_from_zero_iter, n_element, sort_buffer_ptr);
  38: 
  39: 
  40:     // Sort the input data. The original indices of the data are stored in
  41:     // sort_buffer_ptr
  42:     thrust::sort_by_key(policy, iter_begin, iter_end, sort_buffer_ptr);
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 44-45
```cpp
  44:     // Count # of unique elements via an inner product between adjacent elements.
  45:     // Add 1 if two neighboring element are not equal.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 46-54
```cpp
  46:     int unique = 1 +
  47:         thrust::inner_product(
  48:                     policy,
  49:                     iter_begin,
  50:                     iter_end - 1,
  51:                     iter_begin + 1,
  52:                     0,
  53:                     thrust::plus<int>(),
  54:                     thrust::not_equal_to<scalar_t>());
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 56-56
```cpp
  56:     // Count frequency of each element
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 57-58
```cpp
  57:     auto keys = c10::DeviceArray<scalar_t>(*cuda_allocator, unique);
  58:     auto counts = c10::DeviceArray<int64_t>(*cuda_allocator, unique);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 60-61
```cpp
  60:     auto keys_ptr = thrust::device_pointer_cast(keys.get());
  61:     auto counts_ptr = thrust::device_pointer_cast(counts.get());
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 63-69
```cpp
  63:     thrust::reduce_by_key(
  64:         policy,
  65:         iter_begin,
  66:         iter_end,
  67:         thrust::constant_iterator<int>(1),
  68:         keys_ptr,
  69:         counts_ptr);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 71-71
```cpp
  71:     // Find index of maximum count
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 72-73
```cpp
  72:     auto it = thrust::max_element(policy, counts_ptr, counts_ptr + unique);
  73:     scalar_t mode = keys_ptr[it - counts_ptr];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 75-75
```cpp
  75:     // Find first index within which it occurs
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 76-76
```cpp
  76:     auto position_iter = thrust::find(policy, iter_begin, iter_end, mode);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 78-78
```cpp
  78:     // Translate to original non-sorted index
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 79-83
```cpp
  79:     TORCH_INTERNAL_ASSERT(position_iter != iter_end);
  80:     int64_t index = sort_buffer_ptr[position_iter - iter_begin];
  81:     return {mode, index};
  82:   }
  83: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 85-91
```cpp
  85: struct EqualsMode {
  86:   bool mode;
  87: 
  88:   C10_DEVICE bool operator()(const uint8_t x) {
  89:     return static_cast<bool>(x) == mode;
  90:   }
  91: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 93-114
```cpp
  93: template <>
  94: struct ModeImpl<bool> {
  95:   std::tuple<bool, int64_t> operator()(
  96:       const bool *first,
  97:       const bool *last) {
  98:     at::cuda::ThrustAllocator thrust_allocator;
  99:     auto stream = at::cuda::getCurrentCUDAStream();
 100:     auto policy = thrust::cuda::par(thrust_allocator).on(stream);
 101: 
 102:     // For bool, we can skip finding the unique elements since there
 103:     // are only two possible values.
 104: 
 105:     // See NOTE [Loading boolean values]
 106:     auto first_bytes = reinterpret_cast<const uint8_t*>(first);
 107:     auto last_bytes = reinterpret_cast<const uint8_t*>(last);
 108: 
 109:     const auto numel = last - first;
 110:     const auto num_true = thrust::count_if(
 111:         policy,
 112:         first_bytes,
 113:         last_bytes,
 114:         [] GPU_LAMBDA (uint8_t x) {
```
- EN: This block defines or continues the implementation of `count_if`.
- CN: 该代码块定义或继续实现 `count_if`。

### Lines 115-119
```cpp
 115:           return static_cast<bool>(x);
 116:         }
 117:       );
 118:     const auto num_false = (numel - num_true);
 119:     const auto mode = num_true > num_false;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 121-121
```cpp
 121:     // Find first index within which it occurs
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 122-127
```cpp
 122:     const auto position_iter = thrust::find_if(
 123:         policy, first_bytes, last_bytes, EqualsMode{mode});
 124:     const int64_t index = position_iter - first_bytes;
 125:     return {mode, index};
 126:   }
 127: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 129-150
```cpp
 129: template <typename scalar_t>
 130: void calculate_mode(
 131:     const TensorBase& values,
 132:     const TensorBase& indices,
 133:     const TensorBase& self,
 134:     std::vector<int64_t>& position,
 135:     int dim) {
 136: 
 137:   TORCH_INTERNAL_ASSERT(self.is_contiguous());
 138: 
 139:   // Because the input is contiguous, we want to get a reference to the
 140:   // location of the buffer at the innermost dimension that we are going
 141:   // to calculate the mode for --> we do this by manually doing the stride
 142:   // calculations to get an offset
 143:   //
 144:   // Yes, mutating self is a code smell, but we clone self before
 145:   // entering the bowels of this implementation.
 146:   //
 147:   // See [Note: CUDA torch.mode clones self]
 148:   scalar_t* data = self.mutable_data_ptr<scalar_t>();
 149:   for (int64_t i = 0; i < static_cast<int64_t>(position.size()); i++) {
 150:     data += position[i] * ensure_nonempty_stride(self, i);
```
- EN: This block defines or continues the implementation of `calculate_mode`.
- CN: 该代码块定义或继续实现 `calculate_mode`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 151-151
```cpp
 151:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 153-154
```cpp
 153:   int64_t ndim = ensure_nonempty_dim(self.dim());
 154:   int64_t n_element = ensure_nonempty_size(self, ndim - 1);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 156-157
```cpp
 156:   scalar_t* iter_begin = data;
 157:   scalar_t* iter_end = data + n_element;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 159-159
```cpp
 159:   auto [mode, index] = ModeImpl<scalar_t>{}(iter_begin, iter_end);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 161-161
```cpp
 161:   // Place mode, index in output
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 162-163
```cpp
 162:   scalar_t* values_data = values.mutable_data_ptr<scalar_t>();
 163:   int64_t* indices_data = indices.mutable_data_ptr<int64_t>();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 165-169
```cpp
 165:   for (int64_t i = 0; i < static_cast<int64_t>(position.size()); i++) {
 166:     int64_t pos = position[i];
 167:     values_data += ensure_nonempty_stride(values, i) * pos;
 168:     indices_data += ensure_nonempty_stride(indices, i) * pos;
 169:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 171-176
```cpp
 171:   auto stream = at::cuda::getCurrentCUDAStream();
 172:   AT_CUDA_CHECK(cudaMemcpyAsync(
 173:       values_data, &mode, sizeof(scalar_t), cudaMemcpyHostToDevice, stream));
 174:   //memcpy_and_sync will synchronize results
 175:   at::cuda::memcpy_and_sync(indices_data, &index, sizeof(int64_t), cudaMemcpyHostToDevice, stream);
 176: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 178-197
```cpp
 178: template <typename scalar_t>
 179: void apply_mode(
 180:     const TensorBase& values,
 181:     const TensorBase& indices,
 182:     const TensorBase& self,
 183:     std::vector<int64_t>& position,
 184:     int dim,
 185:     int curDim) {
 186:   // Because we have transposed the Tensor, the data for the dimension we are
 187:   // mode'ing along is always in the innermost dimension
 188:   int64_t ndim = ensure_nonempty_dim(self.dim());
 189:   if (curDim == ndim - 1) {
 190:     calculate_mode<scalar_t>(values, indices, self, position, dim);
 191:   } else {
 192:     for (int i = 0; i < ensure_nonempty_size(self, curDim); ++i) {
 193:       position[curDim] = i;
 194:       apply_mode<scalar_t>(values, indices, self, position, dim, curDim + 1);
 195:     }
 196:   }
 197: }
```
- EN: This block defines or continues the implementation of `apply_mode`.
- CN: 该代码块定义或继续实现 `apply_mode`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 199-217
```cpp
 199: template <int64_t size, typename scalar_t>
 200: void handle_fused_mode(
 201:     dim3 grid,
 202:     const TensorBase& self,
 203:     cuda::detail::TensorInfo<scalar_t, unsigned int>& ti_values,
 204:     cuda::detail::TensorInfo<int64_t, unsigned int>& ti_indices,
 205:     int64_t slice_size,
 206:     int64_t slices) {
 207:   constexpr int num_threads = size / 2;
 208:   int warp_size = at::cuda::warp_size();
 209:   TORCH_INTERNAL_ASSERT(num_threads % warp_size == 0 &&
 210:                 num_threads <= cuda_utils::kCUDABlockReduceMaxThreads(), "");
 211:   const auto memsize =
 212:       (sizeof(scalar_t) * size) + (2 * size * sizeof(unsigned int));
 213:   compute_mode<scalar_t, size>
 214:       <<<grid, num_threads, memsize, at::cuda::getCurrentCUDAStream()>>>(
 215:           self.const_data_ptr<scalar_t>(), ti_values, ti_indices, slice_size, slices);
 216:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 217: }
```
- EN: This block defines or continues the implementation of `handle_fused_mode`.
- CN: 该代码块定义或继续实现 `handle_fused_mode`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 219-240
```cpp
 219: template <typename scalar_t>
 220: void fused_mode(
 221:     const TensorBase& values,
 222:     const TensorBase& indices,
 223:     const TensorBase& self,
 224:     int64_t slice_size,
 225:     int64_t slices) {
 226:   // Set-up TensorInfo structs for passing to kernel
 227:   auto ti_values = cuda::detail::getTensorInfo<scalar_t, unsigned int>(values);
 228:   auto ti_indices = cuda::detail::getTensorInfo<int64_t, unsigned int>(indices);
 229: 
 230:   // The number of blocks is the number of slices that we need to calculate
 231:   // the mode for. Each block is responsible for computing a single mode
 232:   dim3 grid;
 233:   getGridFromTiles(slices, grid);
 234: 
 235:   // The blocksize is two elements per thread, rounded up to the nearest power
 236:   // of 2
 237:   auto ceilPowerOf2 = nextHighestPowerOf2(slice_size);
 238: 
 239:   // Tradeoff between compilation time and the number of specializations.
 240:   // Ideally we would have one handle_fused_mode for each power of 2
```
- EN: This block defines or continues the implementation of `fused_mode`.
- CN: 该代码块定义或继续实现 `fused_mode`。

### Lines 241-262
```cpp
 241:   switch (ceilPowerOf2) {
 242:     case 2048:
 243:       handle_fused_mode<2048, scalar_t>(
 244:           grid, self, ti_values, ti_indices, slice_size, slices);
 245:       break;
 246:     case 1024:
 247:     case 512:
 248:     case 256:
 249:       handle_fused_mode<1024, scalar_t>(
 250:           grid, self, ti_values, ti_indices, slice_size, slices);
 251:       break;
 252:     case 128:
 253:     case 64:
 254:     case 32:
 255:     case 16:
 256:     case 8:
 257:     case 4:
 258:     case 2:
 259:       handle_fused_mode<128, scalar_t>(
 260:           grid, self, ti_values, ti_indices, slice_size, slices);
 261:       break;
 262:     case 1:
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 263-265
```cpp
 263:     default:
 264:       TORCH_INTERNAL_ASSERT(false);
 265:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 267-268
```cpp
 267:   AT_CUDA_CHECK(cudaGetLastError());
 268: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 270-276
```cpp
 270: void launch_fused_mode_kernel(
 271:     const TensorBase &values, const TensorBase &indices, const TensorBase &self,
 272:     int64_t slice_size, int64_t slices) {
 273:   AT_DISPATCH_ALL_TYPES_AND3(kBool, kBFloat16, kHalf, self.scalar_type(), "cuda_mode", [&] {
 274:     fused_mode<scalar_t>(values, indices, self, slice_size, slices);
 275:   });
 276: }
```
- EN: This block defines or continues the implementation of `launch_fused_mode_kernel`.
- CN: 该代码块定义或继续实现 `launch_fused_mode_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 278-286
```cpp
 278: void launch_apply_mode_kernel(const TensorBase &values, const TensorBase &indices,
 279:                               const TensorBase &self, int64_t dim, int64_t ndim) {
 280:   AT_DISPATCH_ALL_TYPES_AND3(kBool, kBFloat16, kHalf, self.scalar_type(), "cuda_mode", [&] {
 281:     // Position will store the dimension values we are processing
 282:     std::vector<int64_t> position(ndim - 1, 0);
 283: 
 284:     apply_mode<scalar_t>(values, indices, self, position, dim, 0);
 285:   });
 286: }
```
- EN: This block defines or continues the implementation of `launch_apply_mode_kernel`.
- CN: 该代码块定义或继续实现 `launch_apply_mode_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 288-288
```cpp
 288: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- Thrust utilities supply STL-like CUDA algorithms and containers. / Thrust 工具提供类似 STL 的 CUDA 算法与容器。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/native/cuda/TensorModeKernel.cuh>`
  - `<ATen/native/cuda/TensorModeKernel.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/native/NonEmptyUtils.h>`
  - `<ATen/cuda/detail/IndexUtils.cuh>`
  - `<ATen/cuda/ThrustAllocator.h>`
  - `<c10/core/DeviceArray.h>`
  - `<thrust/count.h>`
  - `<thrust/device_ptr.h>`
  - `<thrust/device_vector.h>`
  - `<thrust/execution_policy.h>`
  - `<thrust/extrema.h>`
- Runtime symbols / 运行时符号:
  - `launch_fused_mode_kernel`
  - `launch_apply_mode_kernel`
  - `AT_DISPATCH_ALL_TYPES_AND3`
  - `thrust::cuda::par`
  - `thrust::device_pointer_cast`
  - `thrust::make_counting_iterator`
  - `thrust::copy_n`
  - `thrust::sort_by_key`
  - `thrust::inner_product`
  - `thrust::plus`
  - `thrust::not_equal_to`
  - `thrust::reduce_by_key`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
