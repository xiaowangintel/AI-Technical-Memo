# TensorTransformations.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/TensorTransformations.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `kernel_pointwise_flip_apply2`, `flip_cuda_kernel`, `roll_cuda_kernel`, `roll_cuda`.
- 用途（中文）: 实现与 `kernel_pointwise_flip_apply2`, `flip_cuda_kernel`, `roll_cuda_kernel`, `roll_cuda` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/native/TensorTransformations.h>
   3: 
   4: #include <ATen/Dispatch.h>
   5: #include <ATen/cuda/detail/IndexUtils.cuh>
   6: #include <ATen/cuda/CUDAApplyUtils.cuh>
   7: #include <ATen/cuda/CUDAContext.h>
   8: #include <c10/macros/Macros.h>
   9: 
  10: #ifndef AT_PER_OPERATOR_HEADERS
  11: #include <ATen/Functions.h>
  12: #include <ATen/NativeFunctions.h>
  13: #else
  14: #include <ATen/ops/empty_like.h>
  15: #include <ATen/ops/roll_native.h>
  16: #endif
  17: 
  18: #include <cstddef>
  19: #include <vector>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/TensorTransformations.h>`, `<ATen/Dispatch.h>`, `<ATen/cuda/detail/IndexUtils.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/TensorTransformations.h>`, `<ATen/Dispatch.h>`, `<ATen/cuda/detail/IndexUtils.cuh>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 21-42
```cpp
  21: namespace at::native {
  22: 
  23: template <typename scalar_t, typename IndexType>
  24: C10_LAUNCH_BOUNDS_2(cuda::getApplyBlockSize(), cuda::getApplyBlocksPerSM())
  25: __global__ void kernel_pointwise_flip_apply2(
  26:     const cuda::detail::TensorInfo<scalar_t, IndexType> in_tensor_info,
  27:     cuda::detail::TensorInfo<scalar_t, IndexType> out_tensor_info,
  28:     IndexType N,
  29:     int flip_dim,
  30:     IndexType total_dims) {
  31:   for (IndexType linear_index = blockIdx.x * blockDim.x + threadIdx.x; linear_index < N; linear_index += gridDim.x * blockDim.x) {
  32:     IndexType dst_offset = 0;
  33:     if (flip_dim == 0) {
  34:       // flip 1st dim
  35:       dst_offset = (in_tensor_info.sizes[0] - 1 - linear_index / in_tensor_info.strides[0]) * in_tensor_info.strides[0] + linear_index % in_tensor_info.strides[0];
  36:     }
  37:     else {
  38:       // flip last dim
  39:       IndexType i = total_dims - 1;
  40:       dst_offset = linear_index / in_tensor_info.strides[0] * in_tensor_info.strides[0] + (in_tensor_info.sizes[i] - 1 - linear_index % in_tensor_info.strides[0]);
  41:     }
  42:     out_tensor_info.data[dst_offset] = in_tensor_info.data[linear_index];
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `kernel_pointwise_flip_apply2`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `kernel_pointwise_flip_apply2`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 43-44
```cpp
  43:   }
  44: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 46-67
```cpp
  46: template <typename scalar_t>
  47: C10_LAUNCH_BOUNDS_1(cuda::getApplyBlockSize())
  48: __global__ void flip_cuda_kernel(
  49:     scalar_t* in_tensor,
  50:     scalar_t* out_tensor,
  51:     int64_t N,
  52:     int64_t* flip_dims,
  53:     int64_t flip_dims_size,
  54:     int64_t* strides,
  55:     int64_t* strides_contiguous,
  56:     int64_t* shape,
  57:     int64_t total_dims) {
  58:   int64_t linear_index = ((int64_t) blockIdx.x) * blockDim.x + threadIdx.x;
  59:   if (linear_index >= N) {
  60:     return;
  61:   }
  62: 
  63:   int64_t cur_indices = linear_index, rem = 0, dst_offset = 0;
  64:   for (int64_t i = 0; i < total_dims; i++) {
  65:     int64_t temp = cur_indices;
  66:     cur_indices = cur_indices / strides_contiguous[i];
  67:     rem = temp - cur_indices * strides_contiguous[i];
```
- EN: This block defines GPU kernel entry point(s) `flip_cuda_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `flip_cuda_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 68-68
```cpp
  68:     // flip the indices if it is in flip_dims
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 69-78
```cpp
  69:     for (int64_t j = 0; j < flip_dims_size; j++) {
  70:       if (i == flip_dims[j]) {
  71:         cur_indices = shape[i] - 1 - cur_indices;
  72:       }
  73:     }
  74:     dst_offset += cur_indices * strides[i];
  75:     cur_indices = rem;
  76:   }
  77:   out_tensor[linear_index] = in_tensor[dst_offset];
  78: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 80-80
```cpp
  80: #if defined(USE_ROCM)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 82-103
```cpp
  82: template <typename scalar_t>
  83: C10_LAUNCH_BOUNDS_1(cuda::getApplyBlockSize())
  84: __global__ void roll_cuda_kernel(
  85:     const scalar_t* in_tensor,
  86:     scalar_t* out_tensor,
  87:     int64_t N,
  88:     int64_t roll_dim,
  89:     int64_t start,
  90:     int64_t size,
  91:     int64_t stride,
  92:     int64_t total_dims) {
  93:   for (int64_t linear_index = ((int64_t) blockIdx.x) * blockDim.x + threadIdx.x;
  94:        linear_index < N; linear_index += blockDim.x*gridDim.x)
  95:   {
  96:     // roll dim idx is the index of linear_index along the rolling dimension.
  97:     int64_t roll_dim_idx = linear_index % (stride * size) / stride;
  98:     // index into the source data to find appropriate value.
  99:     int64_t source_idx = 0;
 100:     if( roll_dim_idx >= (size - start) ) {
 101:       source_idx = linear_index - ((size - start) * stride);
 102:     } else {
 103:       source_idx = linear_index + (start * stride);
```
- EN: This block defines GPU kernel entry point(s) `roll_cuda_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `roll_cuda_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 104-107
```cpp
 104:     }
 105:     out_tensor[linear_index] = in_tensor[source_idx];
 106:   }
 107: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 109-109
```cpp
 109: #else
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 111-132
```cpp
 111: template <typename scalar_t>
 112: C10_LAUNCH_BOUNDS_1(cuda::getApplyBlockSize())
 113: __global__ void roll_cuda_kernel(
 114:     const scalar_t* in_tensor,
 115:     scalar_t* out_tensor,
 116:     int64_t N,
 117:     int64_t roll_dim,
 118:     int64_t start,
 119:     int64_t size,
 120:     int64_t stride,
 121:     int64_t total_dims) {
 122:   int64_t linear_index = ((int64_t) blockIdx.x) * blockDim.x + threadIdx.x;
 123:   if (linear_index >= N) {
 124:     return;
 125:   }
 126:   // roll dim idx is the index of linear_index along the rolling dimension.
 127:   int64_t roll_dim_idx = linear_index % (stride * size) / stride;
 128:   // index into the source data to find appropriate value.
 129:   int64_t source_idx = 0;
 130:   if( roll_dim_idx >= (size - start) ) {
 131:     source_idx = linear_index - ((size - start) * stride);
 132:   } else {
```
- EN: This block defines GPU kernel entry point(s) `roll_cuda_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `roll_cuda_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 133-136
```cpp
 133:     source_idx = linear_index + (start * stride);
 134:   }
 135:   out_tensor[linear_index] = in_tensor[source_idx];
 136: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 138-138
```cpp
 138: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 140-140
```cpp
 140: // Roll a tensor along a dimension
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 141-162
```cpp
 141: Tensor roll_cuda(const Tensor& self, IntArrayRef shifts, IntArrayRef dims) {
 142:   if (dims.size() != 1 || shifts.size() != 1) {
 143:     return roll_common(self, shifts, dims);
 144:   }
 145: 
 146:   auto in_tensor = self;
 147:   if(!self.is_contiguous()) {
 148:     in_tensor = self.contiguous();
 149:   }
 150:   auto out_tensor = at::empty_like(in_tensor, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
 151:   if (out_tensor.numel() == 0) {
 152:     return out_tensor;
 153:   }
 154:   const int64_t N = in_tensor.numel();
 155:   const int64_t dim = dims[0];
 156:   const int64_t size = in_tensor.size(dim);
 157:   int64_t start = (size - shifts[0]) % size;
 158:   // Behavior of % is different in C++ vs Python for negative numbers. This
 159:   // corrects the difference.
 160:   if( start < 0 ) start = start + size;
 161: 
 162:   dim3 dim_block = cuda::getApplyBlock();
```
- EN: This block defines or continues the implementation of `roll_cuda`.
- CN: 该代码块定义或继续实现 `roll_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 163-163
```cpp
 163: #if defined(USE_ROCM)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 164-170
```cpp
 164:   const int num_mp = at::cuda::getCurrentDeviceProperties()->multiProcessorCount;
 165:   // Given a thread block size of 512, we launch with 4 blocks per SM/CU
 166:   dim3 dim_grid(num_mp * 4);
 167: #else
 168:   dim3 dim_grid;
 169:   TORCH_CHECK(cuda::getApplyGrid(N, dim_grid, in_tensor.get_device()), "unable to get dim grid");
 170: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 172-172
```cpp
 172:   auto total_dims = in_tensor.dim();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 174-186
```cpp
 174:   AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND4(
 175:       at::ScalarType::Half, at::ScalarType::Bool, at::ScalarType::BFloat16,
 176:       at::ScalarType::ComplexHalf,
 177:       in_tensor.scalar_type(), "roll_cuda",
 178:       [&] {
 179:         roll_cuda_kernel<<<dim_grid, dim_block, 0, at::cuda::getCurrentCUDAStream()>>>(
 180:           in_tensor.const_data_ptr<scalar_t>(), out_tensor.mutable_data_ptr<scalar_t>(), N,
 181:           dim, start,
 182:           size,
 183:           in_tensor.stride(dim),
 184:           total_dims);
 185:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 186:       });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 188-189
```cpp
 188:   return out_tensor;
 189: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 191-191
```cpp
 191: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/native/TensorTransformations.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/cuda/detail/IndexUtils.cuh>`
  - `<ATen/cuda/CUDAApplyUtils.cuh>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<c10/macros/Macros.h>`
  - `<ATen/Functions.h>`
  - `<ATen/NativeFunctions.h>`
  - `<ATen/ops/empty_like.h>`
  - `<ATen/ops/roll_native.h>`
  - `<cstddef>`
  - `<vector>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND4`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
