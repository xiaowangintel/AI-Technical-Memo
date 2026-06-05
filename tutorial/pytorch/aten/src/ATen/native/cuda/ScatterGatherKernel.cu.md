# ScatterGatherKernel.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/ScatterGatherKernel.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `_launch_scatter_gather_kernel`, `constexpr`, `gather_cuda_kernel`, `scatter_cuda_kernel`.
- 用途（中文）: 实现与 `_launch_scatter_gather_kernel`, `constexpr`, `gather_cuda_kernel`, `scatter_cuda_kernel` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/native/TensorAdvancedIndexing.h>
   3: #include <ATen/core/Tensor.h>
   4: #include <ATen/Dispatch.h>
   5: #include <ATen/ceil_div.h>
   6: #include <ATen/MemoryOverlap.h>
   7: 
   8: #include <ATen/native/ScatterGatherChecks.h>
   9: #include <ATen/native/ReduceOpsUtils.h>
  10: #include <ATen/native/cuda/IndexKernelUtils.h>
  11: #include <ATen/native/cuda/Loops.cuh>
  12: #include <ATen/native/cuda/KernelUtils.cuh>
  13: #include <ATen/native/cuda/MemoryAccess.cuh>
  14: #include <ATen/cuda/detail/OffsetCalculator.cuh>
  15: #include <ATen/cuda/Atomic.cuh>
  16: #include <ATen/cuda/CUDAContext.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/TensorAdvancedIndexing.h>`, `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/TensorAdvancedIndexing.h>`, `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`。

### Lines 18-39
```cpp
  18: namespace at::native {
  19: 
  20: // Implement as functors since lambdas don't get optimized.
  21: class ReduceMultiply {
  22: public:
  23:   template <typename scalar_t>
  24:   constexpr C10_DEVICE void operator() (scalar_t* self_data_start, int64_t index, int64_t numel, const scalar_t * src_data) const {
  25:     (void)numel; // suppress unused warning
  26:     gpuAtomicMul(self_data_start + index, *src_data);
  27:   }
  28: };
  29: static ReduceMultiply reduce_multiply;
  30: 
  31: class ReduceAdd {
  32: public:
  33:   template <typename scalar_t>
  34:   constexpr C10_DEVICE void operator() (scalar_t* self_data_start, int64_t index, int64_t numel, const scalar_t * src_data) const {
  35: #if (defined(__gfx942__) || defined(__gfx950__))
  36:     opportunistic_fastAtomicAdd(self_data_start, index, numel, *src_data);
  37: #else
  38:     fastAtomicAdd(self_data_start, index, numel, *src_data, true);
  39: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 40-42
```cpp
  40:   }
  41: };
  42: static ReduceAdd reduce_add;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 44-51
```cpp
  44: class ReduceMean {
  45: public:
  46:   template <typename scalar_t>
  47:   constexpr C10_DEVICE void operator() (scalar_t* self_data_start, int64_t index, int64_t numel, const scalar_t * src_data) const {
  48:     fastAtomicAdd(self_data_start, index, numel, *src_data, true);
  49:   }
  50: };
  51: static ReduceMean reduce_mean;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 53-61
```cpp
  53: class ReduceMinimum {
  54: public:
  55:   template <typename scalar_t>
  56:   constexpr C10_DEVICE void operator() (scalar_t* self_data_start, int64_t index, int64_t numel, const scalar_t * src_data) const {
  57:     (void)numel; // suppress unused warning
  58:     gpuAtomicMin(self_data_start + index, *src_data);
  59:   }
  60: };
  61: static ReduceMinimum reduce_minimum;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 63-71
```cpp
  63: class ReduceMaximum {
  64: public:
  65:   template <typename scalar_t>
  66:   constexpr C10_DEVICE void operator() (scalar_t* self_data_start, int64_t index, int64_t numel, const scalar_t * src_data) const {
  67:     (void)numel; // suppress unused warning
  68:     gpuAtomicMax(self_data_start + index, *src_data);
  69:   }
  70: };
  71: static ReduceMaximum reduce_maximum;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 73-81
```cpp
  73: class TensorAssign {
  74: public:
  75:   template <typename scalar_t>
  76:   constexpr C10_DEVICE void operator() (scalar_t* self_data_start, int64_t index, int64_t numel, const scalar_t * src_data) const {
  77:     (void)numel; // suppress unused warning
  78:     *(self_data_start + index) = *src_data;
  79:   }
  80: };
  81: static TensorAssign tensor_assign;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 83-86
```cpp
  83: // The kernels are implemented on an opaque,
  84: // self-aligned type of the correct size,
  85: // to avoid redundant kernels for different types
  86: // of the same size.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 87-87
```cpp
  87: template <int N> struct alignas(N) OpaqueType { char data[N]; };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 89-89
```cpp
  89: // essentially rewritten related to legacy::launch_kernel parts
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 90-103
```cpp
  90: template <int nt, int vt, typename func_t>
  91: C10_LAUNCH_BOUNDS_2(nt, vt)
  92: __global__ void _scatter_gather_elementwise_kernel(int N, func_t f) {
  93:   constexpr int nv = nt * vt;
  94:   int idx = nv * blockIdx.x + threadIdx.x;
  95: 
  96:   #pragma unroll
  97:   for (int i = 0; i < vt; ++i) {
  98:     if (idx < N) {
  99:       f(idx);
 100:       idx += nt;
 101:     }
 102:   }
 103: }
```
- EN: This block defines GPU kernel entry point(s) `_scatter_gather_elementwise_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `_scatter_gather_elementwise_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 105-117
```cpp
 105: template <int nt, int vt, typename func_t>
 106: static void _launch_scatter_gather_kernel(int64_t N, const func_t& f) {
 107:   TORCH_INTERNAL_ASSERT(N >= 0 && N <= std::numeric_limits<int32_t>::max());
 108:   if (N == 0) {
 109:     return;
 110:   }
 111: 
 112:   const dim3 block(nt);
 113:   const dim3 grid((N + block.x * vt - 1) / (block.x * vt));
 114:   const auto stream = at::cuda::getCurrentCUDAStream();
 115:   _scatter_gather_elementwise_kernel<nt, vt, func_t><<<grid, block, 0, stream>>>(N, f);
 116:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 117: }
```
- EN: This block defines or continues the implementation of `_launch_scatter_gather_kernel`.
- CN: 该代码块定义或继续实现 `_launch_scatter_gather_kernel`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 119-140
```cpp
 119: template <bool is_scatter_like, typename scalar_t, typename index_t>
 120: struct _cuda_scatter_gather_internal_kernel {
 121:   template <typename func_t>
 122:   void operator() (
 123:     TensorIterator& iter,
 124:     int64_t index_size,
 125:     int64_t index_stride,
 126:     int64_t numel,  // Do not use `const` qualifier here as it may cause issue in cuda 11.6.x. See #75434, #75545
 127:     const func_t& f
 128:   ) {
 129:     if (!iter.can_use_32bit_indexing()) {
 130:       for (auto& sub_iter : iter.with_32bit_indexing()) {
 131:         _cuda_scatter_gather_internal_kernel<is_scatter_like, scalar_t, index_t>()(
 132:           sub_iter, index_size, index_stride, numel, f
 133:         );
 134:       }
 135:       return;
 136:     }
 137: 
 138:     char* self_ptr = (char*)iter.data_ptr(0);
 139:     char* src_ptr = (char*)iter.data_ptr(1);
 140:     char* index_ptr = (char*)iter.data_ptr(2);
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 142-157
```cpp
 142:     if constexpr (!is_scatter_like) {
 143:       // we can go to faster path if we are indexing on the first dim
 144:       // the dst and src are contiguous and all the dims and pts are multiple of 16
 145:       constexpr size_t element_size = sizeof(scalar_t);
 146:       constexpr size_t alignment = 16;
 147:       if (at::native::fast_gather_kernel_eligible<alignment>(iter, self_ptr, src_ptr, index_stride * element_size, element_size)) {
 148:         auto slice_size = iter.shape()[0] * element_size;
 149:         auto num_ind = iter.shape()[1];
 150:         auto ind_dim_size = index_size;
 151:         auto inp_stride_bytes = index_stride * element_size;
 152:         auto out_stride_bytes = iter.strides(0)[1];
 153:         if (iter.numel() == 0) return;
 154:         at::native::vectorized_gather_kernel_launch<alignment, index_t>(self_ptr, src_ptr, (index_t*)index_ptr, num_ind, slice_size, ind_dim_size, inp_stride_bytes, out_stride_bytes);
 155:         return;
 156:       }
 157:     }
```
- EN: This block defines or continues the implementation of `constexpr`.
- CN: 该代码块定义或继续实现 `constexpr`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 158-172
```cpp
 158:     auto offset_calc = make_offset_calculator<3>(iter);
 159:     auto loop = [=]C10_DEVICE(int i) {
 160:       auto offsets = offset_calc.get(i);
 161: 
 162:       int64_t idx_dim = *(index_t*)(index_ptr + offsets[2]);
 163:       CUDA_KERNEL_ASSERT_VERBOSE(idx_dim >= 0 && idx_dim < index_size
 164:         && "scatter gather kernel index out of bounds", "Expected 0 <= idx_dim < index_size (%ld), but got idx_dim = %ld", index_size, idx_dim);
 165: 
 166:       f(
 167:         (scalar_t*)(self_ptr + offsets[0]),
 168:         is_scatter_like ? idx_dim * index_stride : 0,
 169:         numel,
 170:         (scalar_t*)(src_ptr + offsets[1]) + (is_scatter_like ? 0 : idx_dim * index_stride)
 171:       );
 172:     };
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 174-174
```cpp
 174:     _launch_scatter_gather_kernel<num_threads(), thread_work_size()>(iter.numel(), loop);
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 176-177
```cpp
 176:   }
 177: }; // struct _cuda_scatter_gather_internal_kernel
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 179-200
```cpp
 179: template <bool is_scatter_like = true, bool cast_to_opaque = true>
 180: struct cuda_scatter_gather_base_kernel {
 181:   void operator()(
 182:     const Tensor& self, int64_t dim,
 183:     const Tensor& index, const Tensor& src,
 184:     const std::string& method_name,
 185:     const ReduceAdd& f
 186:   ) {
 187:     at::assert_no_internal_overlap(self);
 188: 
 189:     auto index_sizes = ensure_nonempty_vec(index.sizes().vec());
 190:     auto self_strides = ensure_nonempty_vec(self.strides().vec());
 191:     auto src_strides = ensure_nonempty_vec(src.strides().vec());
 192: 
 193:     // restride self and src such that
 194:     // self.shape = src.shape = index.shape
 195:     //
 196:     // restride stride[dim] such that
 197:     // if (is_scatter_like) self.stride[dim] = 0
 198:     // else src.stride[dim] = 0
 199:     auto self_restrided = is_scatter_like ?
 200:         restride_dim(self, dim, index_sizes)
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 201-204
```cpp
 201:       : self.as_strided(index_sizes, self_strides);
 202:     auto src_restrided = is_scatter_like ?
 203:         src.as_strided(index_sizes, src_strides)
 204:       : restride_dim(src, dim, index_sizes);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 206-213
```cpp
 206:     auto iter = TensorIteratorConfig()
 207:       .set_check_mem_overlap(false)
 208:       .check_all_same_dtype(false)
 209:       .resize_outputs(false)
 210:       .add_output(self_restrided)
 211:       .add_const_input(src_restrided)
 212:       .add_const_input(index)
 213:       .build();
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 215-216
```cpp
 215:     auto self_dim_stride = ensure_nonempty_stride(self, dim);
 216:     auto self_dim_size = ensure_nonempty_size(self, dim);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 218-219
```cpp
 218:     auto src_dim_stride = ensure_nonempty_stride(src, dim);
 219:     auto src_dim_size = ensure_nonempty_size(src, dim);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 221-222
```cpp
 221:     auto index_size = is_scatter_like ? self_dim_size : src_dim_size;
 222:     auto index_stride = is_scatter_like ? self_dim_stride : src_dim_stride;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 225-238
```cpp
 225:     AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(
 226:       at::ScalarType::Half, at::ScalarType::Bool, at::ScalarType::BFloat16,
 227:       iter.dtype(),
 228:       "cuda_scatter_gather_base_kernel_func", [&] {
 229:         using dtype = typename std::conditional<cast_to_opaque,
 230:           OpaqueType<sizeof(scalar_t)>, scalar_t>::type;
 231: 
 232:         AT_DISPATCH_INDEX_TYPES(index.scalar_type(), "cuda_scatter_gather_base_kernel_func", [&] () {
 233:           _cuda_scatter_gather_internal_kernel<is_scatter_like, dtype, index_t>()(
 234:             iter, index_size, index_stride, self.numel(), f
 235:           );
 236:         });
 237:       }
 238:     );
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 239-239
```cpp
 239:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 241-262
```cpp
 241:   void operator()(
 242:     const Tensor& self, int64_t dim,
 243:     const Tensor& index, const Tensor& src,
 244:     const std::string& method_name,
 245:     const TensorAssign& f
 246:   ) {
 247:     at::assert_no_internal_overlap(self);
 248: 
 249:     auto index_sizes = ensure_nonempty_vec(index.sizes().vec());
 250:     auto self_strides = ensure_nonempty_vec(self.strides().vec());
 251:     auto src_strides = ensure_nonempty_vec(src.strides().vec());
 252: 
 253:     // restride self and src such that
 254:     // self.shape = src.shape = index.shape
 255:     //
 256:     // restride stride[dim] such that
 257:     // if (is_scatter_like) self.stride[dim] = 0
 258:     // else src.stride[dim] = 0
 259:     auto self_restrided = is_scatter_like ?
 260:         restride_dim(self, dim, index_sizes)
 261:       : self.as_strided(index_sizes, self_strides);
 262:     auto src_restrided = is_scatter_like ?
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 263-264
```cpp
 263:         src.as_strided(index_sizes, src_strides)
 264:       : restride_dim(src, dim, index_sizes);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 266-273
```cpp
 266:     auto iter = TensorIteratorConfig()
 267:       .set_check_mem_overlap(false)
 268:       .check_all_same_dtype(false)
 269:       .resize_outputs(false)
 270:       .add_output(self_restrided)
 271:       .add_const_input(src_restrided)
 272:       .add_const_input(index)
 273:       .build();
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 275-276
```cpp
 275:     auto self_dim_stride = ensure_nonempty_stride(self, dim);
 276:     auto self_dim_size = ensure_nonempty_size(self, dim);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 278-279
```cpp
 278:     auto src_dim_stride = ensure_nonempty_stride(src, dim);
 279:     auto src_dim_size = ensure_nonempty_size(src, dim);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 281-282
```cpp
 281:     auto index_size = is_scatter_like ? self_dim_size : src_dim_size;
 282:     auto index_stride = is_scatter_like ? self_dim_stride : src_dim_stride;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 284-305
```cpp
 284:     if (self.is_quantized()) {
 285:       TORCH_CHECK(
 286:           self.qscheme() == kPerTensorAffine,
 287:           "Only per_tensor quantized quantized tensors are supported by gather.")
 288:       AT_DISPATCH_QINT_TYPES(iter.dtype(), "gather_quant_cuda", [&] {
 289:         using dtype = typename std::conditional<cast_to_opaque,
 290:             OpaqueType<sizeof(scalar_t)>, scalar_t>::type;
 291:         AT_DISPATCH_INDEX_TYPES(index.scalar_type(), "cuda_scatter_gather_base_kernel_func", [&] () {
 292:           _cuda_scatter_gather_internal_kernel<is_scatter_like, dtype, index_t>()(
 293:             iter, index_size, index_stride, self.numel(), f
 294:           );
 295:         });
 296:       });
 297:     } else {
 298:       AT_DISPATCH_V2(
 299:           iter.dtype(),
 300:           "gather_cuda",
 301:           AT_WRAP([&] {
 302:             using dtype = typename std::conditional<cast_to_opaque,
 303:                 OpaqueType<sizeof(scalar_t)>, scalar_t>::type;
 304:             AT_DISPATCH_INDEX_TYPES(index.scalar_type(), "cuda_scatter_gather_base_kernel_func", [&] () {
 305:               _cuda_scatter_gather_internal_kernel<is_scatter_like, dtype, index_t>()(
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 306-318
```cpp
 306:                 iter, index_size, index_stride, self.numel(), f
 307:               );
 308:             });
 309:           }),
 310:           AT_EXPAND(AT_ALL_TYPES_AND_COMPLEX),
 311:           AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES),
 312:           AT_EXPAND(AT_FLOAT8_TYPES),
 313:           kComplexHalf,
 314:           kHalf,
 315:           kBool,
 316:           kBFloat16);
 317:     }
 318:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 320-341
```cpp
 320:   template <typename func_t>
 321:   void operator()(
 322:     const Tensor& self, int64_t dim,
 323:     const Tensor& index, const Tensor& src,
 324:     const std::string& method_name,
 325:     const func_t& f
 326:   ) {
 327:     at::assert_no_internal_overlap(self);
 328: 
 329:     auto index_sizes = ensure_nonempty_vec(index.sizes().vec());
 330:     auto self_strides = ensure_nonempty_vec(self.strides().vec());
 331:     auto src_strides = ensure_nonempty_vec(src.strides().vec());
 332: 
 333:     // restride self and src such that
 334:     // self.shape = src.shape = index.shape
 335:     //
 336:     // restride stride[dim] such that
 337:     // if (is_scatter_like) self.stride[dim] = 0
 338:     // else src.stride[dim] = 0
 339:     auto self_restrided = is_scatter_like ?
 340:         restride_dim(self, dim, index_sizes)
 341:       : self.as_strided(index_sizes, self_strides);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 342-344
```cpp
 342:     auto src_restrided = is_scatter_like ?
 343:         src.as_strided(index_sizes, src_strides)
 344:       : restride_dim(src, dim, index_sizes);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 346-353
```cpp
 346:     auto iter = TensorIteratorConfig()
 347:       .set_check_mem_overlap(false)
 348:       .check_all_same_dtype(false)
 349:       .resize_outputs(false)
 350:       .add_output(self_restrided)
 351:       .add_const_input(src_restrided)
 352:       .add_const_input(index)
 353:       .build();
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 355-356
```cpp
 355:     auto self_dim_stride = ensure_nonempty_stride(self, dim);
 356:     auto self_dim_size = ensure_nonempty_size(self, dim);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 358-359
```cpp
 358:     auto src_dim_stride = ensure_nonempty_stride(src, dim);
 359:     auto src_dim_size = ensure_nonempty_size(src, dim);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 361-362
```cpp
 361:     auto index_size = is_scatter_like ? self_dim_size : src_dim_size;
 362:     auto index_stride = is_scatter_like ? self_dim_stride : src_dim_stride;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 364-377
```cpp
 364:     AT_DISPATCH_ALL_TYPES_AND2(
 365:       at::ScalarType::Half, at::ScalarType::BFloat16,
 366:       iter.dtype(),
 367:       "cuda_scatter_gather_base_kernel_func", [&] {
 368:         using dtype = typename std::conditional<cast_to_opaque,
 369:           OpaqueType<sizeof(scalar_t)>, scalar_t>::type;
 370: 
 371:         AT_DISPATCH_INDEX_TYPES(index.scalar_type(), "cuda_scatter_gather_base_kernel_func", [&] () {
 372:           _cuda_scatter_gather_internal_kernel<is_scatter_like, dtype, index_t>()(
 373:             iter, index_size, index_stride, self.numel(), f
 374:           );
 375:         });
 376:       }
 377:     );
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 378-379
```cpp
 378:   }
 379: }; // struct cuda_scatter_gather_base_kernel
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 381-402
```cpp
 381: template <typename scalar_t, typename index_t>
 382: struct _cuda_scatter_fill_internal_kernel {
 383:   template <typename func_t>
 384:   void operator()(
 385:     TensorIterator& iter,
 386:     scalar_t src_val,
 387:     int64_t index_size,
 388:     int64_t index_stride,
 389:     int64_t numel,  // Do not use `const` qualifier here as it may cause issue in cuda 11.6.x. See #75434, #75545
 390:     const func_t& f
 391:   ) {
 392:     if (!iter.can_use_32bit_indexing()) {
 393:       for (auto& sub_iter : iter.with_32bit_indexing()) {
 394:         _cuda_scatter_fill_internal_kernel<scalar_t, index_t>()(
 395:           sub_iter, src_val, index_size, index_stride, numel, f
 396:         );
 397:       }
 398:       return;
 399:     }
 400: 
 401:     char* self_ptr = (char*)iter.data_ptr(0);
 402:     char* index_ptr = (char*)iter.data_ptr(1);
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 404-418
```cpp
 404:     auto offset_calc = make_offset_calculator<2>(iter);
 405:     auto loop = [=]C10_DEVICE(int i) {
 406:       auto offsets = offset_calc.get(i);
 407: 
 408:       int64_t idx_dim = *(index_t*)(index_ptr + offsets[1]);
 409:       CUDA_KERNEL_ASSERT_VERBOSE(idx_dim >= 0 && idx_dim < index_size
 410:         && "index out of bounds", "Expected 0 <= idx_dim < index_size (%ld), but got idx_dim = %ld", index_size, idx_dim);
 411: 
 412:       f(
 413:         (scalar_t*)(self_ptr + offsets[0]),
 414:         idx_dim * index_stride,
 415:         numel,
 416:         (scalar_t*)&src_val
 417:       );
 418:     };
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 420-422
```cpp
 420:     _launch_scatter_gather_kernel<num_threads(), thread_work_size()>(iter.numel(), loop);
 421:   }
 422: }; // struct _cuda_scatter_fill_internal_kernel
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 424-445
```cpp
 424: template <bool cast_to_opaque = true>
 425: struct cuda_scatter_fill_base_kernel {
 426:   template <typename func_t>
 427:   void operator()(
 428:     const Tensor& self, int64_t dim,
 429:     const Tensor& index, Scalar src,
 430:     const std::string& method_name,
 431:     const func_t& f
 432:   ) {
 433:     at::assert_no_internal_overlap(self);
 434: 
 435:     auto index_sizes = ensure_nonempty_vec(index.sizes().vec());
 436: 
 437:     // restride self such that
 438:     // self.shape = index.shape and
 439:     // self.stride[dim] = 0
 440:     auto self_restrided = restride_dim(self, dim, index_sizes);
 441: 
 442:     auto iter = TensorIteratorConfig()
 443:       .set_check_mem_overlap(false)
 444:       .check_all_same_dtype(false)
 445:       .resize_outputs(false)
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 446-448
```cpp
 446:       .add_output(self_restrided)
 447:       .add_const_input(index)
 448:       .build();
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 450-451
```cpp
 450:     auto index_size = ensure_nonempty_size(self, dim);
 451:     auto index_stride = ensure_nonempty_stride(self, dim);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 453-468
```cpp
 453:     AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(
 454:       at::ScalarType::Half, at::ScalarType::Bool, at::ScalarType::BFloat16,
 455:       iter.dtype(),
 456:       "cuda_scatter_fill_base_kernel_func", [&] {
 457:         using dtype = typename std::conditional<cast_to_opaque,
 458:           OpaqueType<sizeof(scalar_t)>, scalar_t>::type;
 459: 
 460:         auto src_scalar_val = src.to<scalar_t>();
 461:         auto src_val = *(dtype*)&src_scalar_val;
 462: 
 463:         AT_DISPATCH_INDEX_TYPES(index.scalar_type(), "cuda_scatter_fill_base_kernel_func", [&] () {
 464:           _cuda_scatter_fill_internal_kernel<dtype, index_t>()(
 465:             iter, src_val, index_size, index_stride, self.numel(), f
 466:           );
 467:         });
 468:       }
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 469-470
```cpp
 469:     );
 470:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 472-493
```cpp
 472:   void operator()(
 473:     const Tensor& self, int64_t dim,
 474:     const Tensor& index, Scalar src,
 475:     const std::string& method_name,
 476:     const ReduceMultiply& f
 477:   ) {
 478:     at::assert_no_internal_overlap(self);
 479: 
 480:     auto index_sizes = ensure_nonempty_vec(index.sizes().vec());
 481: 
 482:     // restride self such that
 483:     // self.shape = index.shape and
 484:     // self.stride[dim] = 0
 485:     auto self_restrided = restride_dim(self, dim, index_sizes);
 486: 
 487:     auto iter = TensorIteratorConfig()
 488:       .set_check_mem_overlap(false)
 489:       .check_all_same_dtype(false)
 490:       .resize_outputs(false)
 491:       .add_output(self_restrided)
 492:       .add_const_input(index)
 493:       .build();
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 495-496
```cpp
 495:     auto index_size = ensure_nonempty_size(self, dim);
 496:     auto index_stride = ensure_nonempty_stride(self, dim);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 498-513
```cpp
 498:     AT_DISPATCH_ALL_TYPES_AND2(
 499:       at::ScalarType::Half, at::ScalarType::BFloat16,
 500:       iter.dtype(),
 501:       "cuda_scatter_fill_base_kernel_reduce_multiply", [&] {
 502:         using dtype = typename std::conditional<cast_to_opaque,
 503:           OpaqueType<sizeof(scalar_t)>, scalar_t>::type;
 504: 
 505:         auto src_scalar_val = src.to<scalar_t>();
 506:         auto src_val = *(dtype*)&src_scalar_val;
 507: 
 508:         AT_DISPATCH_INDEX_TYPES(index.scalar_type(), "cuda_scatter_fill_base_kernel_reduce_multiply", [&] () {
 509:           _cuda_scatter_fill_internal_kernel<dtype, index_t>()(
 510:             iter, src_val, index_size, index_stride, self.numel(), f
 511:           );
 512:         });
 513:       }
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 514-516
```cpp
 514:     );
 515:   }
 516: }; // struct cuda_scatter_fill_base_kernel
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 518-522
```cpp
 518: void gather_cuda_kernel(const Tensor& result, const Tensor& self, int64_t dim, const Tensor& index) {
 519:   cuda_scatter_gather_base_kernel</*is_scatter_like=*/false>()(
 520:     result, dim, index, self,
 521:     "gather_out_cuda", tensor_assign);
 522: }
```
- EN: This block defines or continues the implementation of `gather_cuda_kernel`.
- CN: 该代码块定义或继续实现 `gather_cuda_kernel`。

### Lines 524-530
```cpp
 524: void scatter_cuda_kernel(const Tensor& self, int64_t dim, const Tensor& index, const Tensor& src) {
 525:   // When indices are not unique, the behavior is non-deterministic
 526:   globalContext().alertNotDeterministic("scatter_cuda_");
 527:   cuda_scatter_gather_base_kernel<>()(
 528:     self, dim, index, src,
 529:     "scatter_cuda_", tensor_assign);
 530: }
```
- EN: This block defines or continues the implementation of `scatter_cuda_kernel`.
- CN: 该代码块定义或继续实现 `scatter_cuda_kernel`。

### Lines 532-536
```cpp
 532: void scatter_fill_cuda_kernel(const Tensor& self, int64_t dim, const Tensor& index, const Scalar& src) {
 533:   cuda_scatter_fill_base_kernel<>()(
 534:     self, dim, index, src,
 535:     "scatter_fill_cuda_", tensor_assign);
 536: }
```
- EN: This block defines or continues the implementation of `scatter_fill_cuda_kernel`.
- CN: 该代码块定义或继续实现 `scatter_fill_cuda_kernel`。

### Lines 538-542
```cpp
 538: void scatter_add_cuda_kernel(const Tensor& self, int64_t dim, const Tensor& index, const Tensor& src) {
 539:   cuda_scatter_gather_base_kernel</*is_scatter_like=*/true, /*cast_to_opaque=*/false>()(
 540:     self, dim, index, src,
 541:     "scatter_add_cuda_", reduce_add);
 542: }
```
- EN: This block defines or continues the implementation of `scatter_add_cuda_kernel`.
- CN: 该代码块定义或继续实现 `scatter_add_cuda_kernel`。

### Lines 544-561
```cpp
 544: void scatter_reduce_cuda_kernel(const Tensor& self, const int64_t dim, const Tensor& index,
 545:                                const Tensor& src, const ReductionType& reduce) {
 546:   // See Note [Writing Nondeterministic Operations]
 547:   // Nondeterministic because of atomicAdd/AtomicMul usage
 548:   globalContext().alertNotDeterministic("scatter_reduce_cuda_kernel");
 549:   switch (reduce) {
 550:   case ReductionType::SUM :
 551:     cuda_scatter_gather_base_kernel<true, false>()(self, dim, index, src,
 552:                                        "scatter_reduce_cuda_add_", reduce_add);
 553:     break;
 554:   case ReductionType::PROD :
 555:     cuda_scatter_gather_base_kernel<true, false>()(self, dim, index, src,
 556:                                        "scatter_reduce_cuda_multiply_", reduce_multiply);
 557:     break;
 558:   default :
 559:     break;
 560:   }
 561: }
```
- EN: This block defines or continues the implementation of `scatter_reduce_cuda_kernel`.
- CN: 该代码块定义或继续实现 `scatter_reduce_cuda_kernel`。

### Lines 563-584
```cpp
 563: void scatter_reduce_two_cuda_kernel(const Tensor& self, const int64_t dim, const Tensor& index,
 564:                                     const Tensor& src, const ReductionType& reduce) {
 565:   switch (reduce) {
 566:   case ReductionType::SUM :
 567:     cuda_scatter_gather_base_kernel<true, false>()(self, dim, index, src,
 568:             "scatter_reduce_cuda_sum_", reduce_add);
 569:     break;
 570:   case ReductionType::PROD :
 571:     globalContext().alertNotDeterministic("scatter_reduce_cuda_prod_");
 572:     cuda_scatter_gather_base_kernel<true, false>()(self, dim, index, src,
 573:             "scatter_reduce_cuda_prod_", reduce_multiply);
 574:     break;
 575:   case ReductionType::MAX :
 576:     cuda_scatter_gather_base_kernel<true, false>()(self, dim, index, src,
 577:             "scatter_reduce_cuda_amax_", reduce_maximum);
 578:     break;
 579:   case ReductionType::MIN :
 580:     cuda_scatter_gather_base_kernel<true, false>()(self, dim, index, src,
 581:             "scatter_reduce_cuda_amin_", reduce_minimum);
 582:     break;
 583:   case ReductionType::MEAN :
 584:     cuda_scatter_gather_base_kernel<true, false>()(self, dim, index, src,
```
- EN: This block defines or continues the implementation of `scatter_reduce_two_cuda_kernel`.
- CN: 该代码块定义或继续实现 `scatter_reduce_two_cuda_kernel`。

### Lines 585-588
```cpp
 585:             "scatter_reduce_cuda_mean_", reduce_mean);
 586:     break;
 587:   }
 588: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 590-604
```cpp
 590: void scatter_scalar_reduce_cuda_kernel(const Tensor& self, const int64_t dim, const Tensor& index,
 591:                                const Scalar& value, const ReductionType& reduce) {
 592:   switch (reduce) {
 593:   case ReductionType::SUM :
 594:     cuda_scatter_fill_base_kernel<false>()(self, dim, index, value,
 595:                                       "scatter_fill_cuda_add_", reduce_add);
 596:     break;
 597:   case ReductionType::PROD :
 598:     cuda_scatter_fill_base_kernel<false>()(self, dim, index, value,
 599:                                       "scatter_fill_cuda_multiply_", reduce_multiply);
 600:     break;
 601:   default :
 602:     break;
 603:   }
 604: }
```
- EN: This block defines or continues the implementation of `scatter_scalar_reduce_cuda_kernel`.
- CN: 该代码块定义或继续实现 `scatter_scalar_reduce_cuda_kernel`。

### Lines 607-613
```cpp
 607: REGISTER_DISPATCH(gather_stub, &gather_cuda_kernel)
 608: REGISTER_DISPATCH(scatter_stub, &scatter_cuda_kernel)
 609: REGISTER_DISPATCH(scatter_fill_stub, &scatter_fill_cuda_kernel)
 610: REGISTER_DISPATCH(scatter_add_stub, &scatter_add_cuda_kernel)
 611: REGISTER_DISPATCH(scatter_reduce_stub, &scatter_reduce_cuda_kernel)
 612: REGISTER_DISPATCH(scatter_scalar_reduce_stub, &scatter_scalar_reduce_cuda_kernel)
 613: REGISTER_DISPATCH(scatter_reduce_two_stub, &scatter_reduce_two_cuda_kernel)
```
- EN: The registration macro binds this implementation into PyTorch dispatch so higher-level operators can call the CUDA specialization.
- CN: 注册宏把该实现接入 PyTorch 分发系统，使上层算子能够调用这个 CUDA 特化版本。

### Lines 615-615
```cpp
 615: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TensorIterator` normalizes tensor shapes/strides and drives elementwise CUDA traversal. / `TensorIterator` 统一张量形状与步长，并驱动逐元素 CUDA 遍历。
- `TensorIteratorConfig` builds iterator state before launch. / `TensorIteratorConfig` 在启动前构建迭代器状态。
- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- Dispatch registration exposes the CUDA specialization to higher PyTorch layers. / 分发注册把 CUDA 特化实现暴露给更高层的 PyTorch 调用链。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/native/TensorAdvancedIndexing.h>`
  - `<ATen/core/Tensor.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/ceil_div.h>`
  - `<ATen/MemoryOverlap.h>`
  - `<ATen/native/ScatterGatherChecks.h>`
  - `<ATen/native/ReduceOpsUtils.h>`
  - `<ATen/native/cuda/IndexKernelUtils.h>`
  - `<ATen/native/cuda/Loops.cuh>`
  - `<ATen/native/cuda/KernelUtils.cuh>`
  - `<ATen/native/cuda/MemoryAccess.cuh>`
  - `<ATen/cuda/detail/OffsetCalculator.cuh>`
- Runtime symbols / 运行时符号:
  - `gather_stub`
  - `scatter_stub`
  - `scatter_fill_stub`
  - `scatter_add_stub`
  - `scatter_reduce_stub`
  - `scatter_scalar_reduce_stub`
  - `scatter_reduce_two_stub`
  - `launch_kernel`
  - `TensorIterator`
  - `TensorIteratorConfig`
  - `AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3`
  - `AT_DISPATCH_INDEX_TYPES`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
