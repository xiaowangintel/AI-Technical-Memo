# IndexKernel.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/IndexKernel.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `launch_kernel`, `gpu_index_kernel`, `index_fill_kernel_impl`, `index_copy_kernel_impl`.
- 用途（中文）: 实现与 `launch_kernel`, `gpu_index_kernel`, `index_fill_kernel_impl`, `index_copy_kernel_impl` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
   1: #define TORCH_ASSERT_NO_OPERATORS
   2: #include <ATen/native/cuda/IndexKernel.h>
   3: #include <ATen/native/IndexKernel.h>
   4: 
   5: #include <array>
   6: #include <type_traits>
   7: #include <ATen/core/TensorBase.h>
   8: #include <ATen/Dispatch.h>
   9: #include <ATen/Dispatch_v2.h>
  10: #include <ATen/cuda/CUDAContext.h>
  11: #include <ATen/cuda/cub.h>
  12: #include <ATen/cuda/detail/IndexUtils.cuh>
  13: #include <ATen/cuda/detail/OffsetCalculator.cuh>
  14: #include <ATen/native/cuda/Loops.cuh>
  15: #include <ATen/native/cuda/KernelUtils.cuh>
  16: #include <ATen/native/quantized/IndexKernel.h>
  17: #include <ATen/native/cuda/MemoryAccess.cuh>
  18: #include <ATen/native/cuda/IndexKernelUtils.h>
  19: 
  20: #include <c10/core/Scalar.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/IndexKernel.h>`, `<ATen/native/IndexKernel.h>`, `<array>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/IndexKernel.h>`, `<ATen/native/IndexKernel.h>`, `<array>`。

### Lines 22-43
```cpp
  22: namespace at::native {
  23: 
  24: static constexpr int launch_bound2 = 4;
  25: 
  26: static constexpr int launch_size_nd = 128;
  27: 
  28: template<int nt, int vt, typename func_t>
  29: C10_LAUNCH_BOUNDS_2(nt, launch_bound2)
  30: __global__ void index_elementwise_kernel(const int64_t N, const func_t f) {
  31:   const auto tid = threadIdx.x;
  32:   const auto nv = nt * vt;
  33:   auto idx = nv * blockIdx.x + tid;
  34:   #pragma unroll
  35:   for (int i = 0; i < vt; i++) {
  36:     if (idx < N) {
  37:       f(idx);
  38:       idx += nt;
  39:     }
  40:   }
  41: }
  42: 
  43: template<int nt, int vt, typename func_t>
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `index_elementwise_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `index_elementwise_kernel`，它们会直接在 CUDA 线程上执行。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 44-54
```cpp
  44: static void launch_kernel(const int64_t N, const func_t& f) {
  45:   TORCH_INTERNAL_ASSERT(N >= 0 && N <= std::numeric_limits<int32_t>::max());
  46:   if (N == 0) {
  47:     return;
  48:   }
  49:   const dim3 block(nt);
  50:   const dim3 grid((N + block.x * vt - 1) / (block.x * vt));
  51:   const auto stream = at::cuda::getCurrentCUDAStream();
  52:   index_elementwise_kernel<nt, vt, func_t><<<grid, block, 0, stream>>>(N, f);
  53:   C10_CUDA_KERNEL_LAUNCH_CHECK();
  54: }
```
- EN: This block defines or continues the implementation of `launch_kernel`.
- CN: 该代码块定义或继续实现 `launch_kernel`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 56-77
```cpp
  56: template <typename func_t>
  57: void gpu_index_kernel(TensorIteratorBase& iter, const IntArrayRef index_size, const IntArrayRef index_stride, const func_t& f, const bool is_gather_like) {
  58:   const auto num_indices = index_size.size();
  59:   AT_ASSERT(num_indices == index_stride.size());
  60:   AT_ASSERT(static_cast<int64_t>(num_indices) == iter.ntensors() - 2);
  61: 
  62:   if (iter.numel() == 0) {
  63:     return;
  64:   }
  65: 
  66:   if (!iter.can_use_32bit_indexing()) {
  67:     for (auto& sub_iter : iter.with_32bit_indexing()) {
  68:       gpu_index_kernel(sub_iter, index_size, index_stride, f, is_gather_like);
  69:     }
  70:     return;
  71:   }
  72: 
  73: 
  74:   char* const out_ptr = static_cast<char*>(iter.data_ptr(0));
  75:   char* const in_ptr = static_cast<char*>(iter.data_ptr(1));
  76:   if (is_gather_like && num_indices==1) {
  77:       const size_t element_size = iter.element_size(0);
```
- EN: This block defines or continues the implementation of `gpu_index_kernel`.
- CN: 该代码块定义或继续实现 `gpu_index_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 78-89
```cpp
  78:       constexpr size_t alignment = 16;
  79:       if (at::native::fast_gather_kernel_eligible<alignment>(iter, out_ptr, in_ptr, index_stride[0], element_size)) {
  80:         auto slice_size = iter.shape()[0] * element_size;
  81:         auto num_ind = iter.shape()[1];
  82:         auto ind_dim_size = index_size[0];
  83:         auto inp_stride_bytes = index_stride[0];
  84:         auto out_stride_bytes = iter.strides(0)[1];
  85:         at::native::vectorized_gather_kernel_launch<alignment, int64_t>(out_ptr, in_ptr, (int64_t*)iter.data_ptr(2), num_ind,
  86:         slice_size, ind_dim_size, inp_stride_bytes, out_stride_bytes, /*allow_neg_indices*/true);
  87:         return;
  88:     }
  89:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 91-98
```cpp
  91:   auto sizes = std::array<int64_t, MAX_DIMS>{};
  92:   auto strides = std::array<int64_t, MAX_DIMS>{};
  93:   auto index_ptrs = std::array<char*, MAX_DIMS>{};
  94:   for (unsigned i = 0; i < num_indices; i++) {
  95:     sizes[i] = index_size[i];
  96:     strides[i] = index_stride[i];
  97:     index_ptrs[i] = (char*)iter.data_ptr(i + 2);
  98:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 101-119
```cpp
 101:   auto offset_calc = make_offset_calculator<3>(iter);
 102:   launch_kernel<launch_size_nd, launch_bound2>(iter.numel(), [=]__device__(int idx) {
 103:     const auto offsets = offset_calc.get(idx);
 104:     char* const out_data = out_ptr + offsets[0];
 105:     const char* const in_data = in_ptr + offsets[1];
 106: 
 107:     int64_t offset = 0;
 108:     #pragma unroll
 109:     for (int i = 0; i < num_indices; i++) {
 110:       int64_t index = *reinterpret_cast<int64_t*>(index_ptrs[i] + offsets[2]);
 111:       CUDA_KERNEL_ASSERT(-sizes[i] <= index && index < sizes[i] && "index out of bounds");
 112:       if (index < 0) {
 113:         index += sizes[i];
 114:       }
 115:       offset += index * strides[i];
 116:     }
 117: 
 118:     f(out_data, in_data, offset);
 119:   });
```
- EN: This block defines or continues the implementation of `numel`.
- CN: 该代码块定义或继续实现 `numel`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 120-120
```cpp
 120: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 122-123
```cpp
 122: // The kernels are templated on an opaque, self-aligned type of the correct
 123: // size to avoid redundant kernels for different types of the same size.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 124-124
```cpp
 124: template <int N> struct alignas(N) OpaqueType { char data[N]; };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 126-147
```cpp
 126: template <typename scalar_t>
 127: void index_fill_kernel_impl(
 128:   TensorIterator& iter,
 129:   const int64_t dim,
 130:   const int64_t self_dim_size,
 131:   const int64_t self_dim_stride,
 132:   const scalar_t fill_val) {
 133:   if (0 == iter.numel()) {
 134:     return;
 135:   }
 136: 
 137:   if (!iter.can_use_32bit_indexing()) {
 138:     for (auto& sub_iter : iter.with_32bit_indexing()) {
 139:       index_fill_kernel_impl(sub_iter, dim, self_dim_size, self_dim_stride, fill_val);
 140:     }
 141:     return;
 142:   }
 143: 
 144:   char* const __restrict__ self_ptr = reinterpret_cast<char*>(iter.data_ptr(0));
 145:   char* const __restrict__ idx_ptr = reinterpret_cast<char*>(iter.data_ptr(1));
 146: 
 147:   const auto offset_calc = make_offset_calculator<2>(iter);
```
- EN: This block defines or continues the implementation of `index_fill_kernel_impl`.
- CN: 该代码块定义或继续实现 `index_fill_kernel_impl`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 149-162
```cpp
 149:   const auto loop = [=]C10_DEVICE(int i) {
 150:     const auto offsets = offset_calc.get(i);
 151: 
 152:     auto* __restrict__ self_data = reinterpret_cast<scalar_t*>(self_ptr + offsets[0]);
 153:     auto idx = *reinterpret_cast<int64_t*>(idx_ptr + offsets[1]);
 154:     CUDA_KERNEL_ASSERT(idx >= -self_dim_size && idx < self_dim_size && "index out of bounds");
 155:     if (idx < 0) {
 156:       idx += self_dim_size;
 157:     }
 158: 
 159:     self_data[idx * self_dim_stride] = fill_val;
 160:   };
 161:   launch_kernel<launch_size_nd, launch_bound2>(iter.numel(), loop);
 162: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 164-185
```cpp
 164: template <typename scalar_t>
 165: void index_copy_kernel_impl(
 166:   TensorIterator& iter,
 167:   const int64_t dim,
 168:   const int64_t self_dim_size,
 169:   const int64_t self_dim_stride) {
 170:   if (iter.numel() == 0) {
 171:     return;
 172:   }
 173: 
 174:   if (!iter.can_use_32bit_indexing()) {
 175:     for (auto& sub_iter : iter.with_32bit_indexing()) {
 176:       index_copy_kernel_impl<scalar_t>(sub_iter, dim, self_dim_size, self_dim_stride);
 177:     }
 178:     return;
 179:   }
 180: 
 181:   char* const __restrict__ self_ptr = reinterpret_cast<char*>(iter.data_ptr(0));
 182:   char* const __restrict__ idx_ptr = reinterpret_cast<char*>(iter.data_ptr(1));
 183:   char* const __restrict__ source_ptr = reinterpret_cast<char*>(iter.data_ptr(2));
 184: 
 185:   const auto offset_calc = make_offset_calculator<3>(iter);
```
- EN: This block defines or continues the implementation of `index_copy_kernel_impl`.
- CN: 该代码块定义或继续实现 `index_copy_kernel_impl`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 187-198
```cpp
 187:   const auto loop = [=]C10_DEVICE(int i) {
 188:     const auto offsets = offset_calc.get(i);
 189: 
 190:     auto* const __restrict__ self_data = reinterpret_cast<scalar_t*>(self_ptr + offsets[0]);
 191:     auto idx = *reinterpret_cast<int64_t*>(idx_ptr + offsets[1]);
 192:     const auto* const __restrict__ source_data = reinterpret_cast<scalar_t*>(source_ptr + offsets[2]);
 193:     CUDA_KERNEL_ASSERT(idx >= 0 && idx < self_dim_size && "index_copy_(): index out of bounds");
 194: 
 195:     self_data[idx * self_dim_stride] = *source_data;
 196:   };
 197:   launch_kernel<launch_size_nd, launch_bound2>(iter.numel(), loop);
 198: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 200-205
```cpp
 200: template <typename scalar_t>
 201: void index_kernel_impl(TensorIteratorBase& iter, const IntArrayRef index_size, const IntArrayRef index_stride) {
 202:   gpu_index_kernel(iter, index_size, index_stride, []C10_DEVICE(char* const out_data, const char* const in_data, const int64_t offset) {
 203:     *reinterpret_cast<scalar_t*>(out_data) = *reinterpret_cast<const scalar_t*>(in_data + offset);
 204:   }, true);
 205: }
```
- EN: This block defines or continues the implementation of `index_kernel_impl`, `gpu_index_kernel`.
- CN: 该代码块定义或继续实现 `index_kernel_impl`, `gpu_index_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 207-212
```cpp
 207: template <typename scalar_t>
 208: void index_put_kernel_impl(TensorIterator& iter, const IntArrayRef index_size, const IntArrayRef index_stride) {
 209:   gpu_index_kernel(iter, index_size, index_stride, []C10_DEVICE(char* const out_data, const char* const in_data, const int64_t offset) {
 210:     *reinterpret_cast<scalar_t*>(out_data + offset) = *reinterpret_cast<const scalar_t*>(in_data);
 211:   }, false);
 212: }
```
- EN: This block defines or continues the implementation of `index_put_kernel_impl`, `gpu_index_kernel`.
- CN: 该代码块定义或继续实现 `index_put_kernel_impl`, `gpu_index_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 214-231
```cpp
 214: static void index_kernel(
 215:     TensorIteratorBase& iter,
 216:     const IntArrayRef index_size,
 217:     const IntArrayRef index_stride) {
 218:   AT_DISPATCH_V2(
 219:       iter.dtype(),
 220:       "index_cuda",
 221:       AT_WRAP([&] {
 222:         using dtype = OpaqueType<sizeof(scalar_t)>;
 223:         index_kernel_impl<dtype>(iter, index_size, index_stride);
 224:       }),
 225:       AT_EXPAND(AT_ALL_TYPES_AND_COMPLEX),
 226:       AT_EXPAND(AT_FLOAT8_TYPES),
 227:       kComplexHalf,
 228:       kHalf,
 229:       kBool,
 230:       kBFloat16);
 231: }
```
- EN: This block defines or continues the implementation of `index_kernel`.
- CN: 该代码块定义或继续实现 `index_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 233-247
```cpp
 233: static void index_fill_kernel(
 234:   TensorIterator& iter,
 235:   const int64_t dim,
 236:   const int64_t self_dim_size,
 237:   const int64_t self_dim_stride,
 238:   const Scalar& source) {
 239:   AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND4(
 240:     at::ScalarType::Half, at::ScalarType::Bool, at::ScalarType::BFloat16, kComplexHalf,
 241:     iter.dtype(), "index_fill_cuda", [&] {
 242:     using dtype = OpaqueType<sizeof(scalar_t)>;
 243:     const auto fill_val = source.to<scalar_t>();
 244:     const auto fill_val_opaque = *reinterpret_cast<const dtype*>(&fill_val);
 245:     index_fill_kernel_impl<dtype>(iter, dim, self_dim_size, self_dim_stride, fill_val_opaque);
 246:   });
 247: }
```
- EN: This block defines or continues the implementation of `index_fill_kernel`.
- CN: 该代码块定义或继续实现 `index_fill_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 249-263
```cpp
 249: static void index_copy_kernel(
 250:   TensorIterator& iter,
 251:   const int64_t dim,
 252:   const int64_t self_dim_size,
 253:   const int64_t self_dim_stride) {
 254:   // See note [Writing Nondeterministic Operations]
 255:   // Nondeterministic when index contains duplicate entries
 256:   // this kernel will not be called when torch.use_deterministic_algorithms(True)
 257:   AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND4(
 258:     at::ScalarType::Half, at::ScalarType::Bool, at::ScalarType::BFloat16, kComplexHalf,
 259:     iter.dtype(), "index_copy_cuda", [&] {
 260:     using dtype = OpaqueType<sizeof(scalar_t)>;
 261:     index_copy_kernel_impl<dtype>(iter, dim, self_dim_size, self_dim_stride);
 262:   });
 263: }
```
- EN: This block defines or continues the implementation of `index_copy_kernel`.
- CN: 该代码块定义或继续实现 `index_copy_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 266-281
```cpp
 266: static void index_put_kernel(TensorIterator& iter, const IntArrayRef index_size, const IntArrayRef index_stride, const bool accumulate) {
 267:   TORCH_CHECK(!accumulate, "index_put does not support accumulate=true");
 268:   AT_DISPATCH_V2(
 269:     iter.dtype(),
 270:     "index_put",
 271:     AT_WRAP([&] {
 272:       using dtype = OpaqueType<sizeof(scalar_t)>;
 273:       index_put_kernel_impl<dtype>(iter, index_size, index_stride);
 274:     }),
 275:     AT_EXPAND(AT_ALL_TYPES_AND_COMPLEX),
 276:     AT_EXPAND(AT_FLOAT8_TYPES),
 277:     kComplexHalf,
 278:     kHalf,
 279:     kBool,
 280:     kBFloat16);
 281: }
```
- EN: This block defines or continues the implementation of `index_put_kernel`.
- CN: 该代码块定义或继续实现 `index_put_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 283-303
```cpp
 283: void index_put_kernel_quantized_cuda(TensorIterator& iter, const IntArrayRef index_size, const IntArrayRef index_stride, const bool accumulate, const double scale, const int zero_point) {
 284:   TORCH_CHECK(!accumulate, "index_put does not support accumulate=true");
 285:   AT_DISPATCH_QINT_AND_SUB_BYTE_TYPES(iter.dtype(), "index_put", [&] {
 286:     constexpr int64_t qmin = std::numeric_limits<typename scalar_t::underlying>::min();
 287:     constexpr int64_t qmax = std::numeric_limits<typename scalar_t::underlying>::max();
 288:     const float inv_scale = 1.0f / static_cast<float>(scale);
 289: 
 290:     gpu_index_kernel(iter, index_size, index_stride, [inv_scale, zero_point, qmin, qmax]C10_DEVICE(char* const out_data, const char* const in_data, const int64_t offset) {
 291:       int64_t qvalue = static_cast<int64_t>(zero_point + nearbyintf(*(float*)in_data * inv_scale));
 292:       // See https://github.com/pytorch/pytorch/issues/127666
 293:       // and https://github.com/pytorch/pytorch/issues/128253.
 294:       // hip-clang std::clamp __glibcxx_assert_fail host function when building on Fedora40/gcc14.
 295:       // The following replaces std::clamp(qvalue, qmin, qmax) and is a viable solution for
 296:       // both CUDA and ROCm since std::clamp and this replacement generates the same PTX.
 297:       // Using #ifdef USE_ROCM to differentiate caused Windows build failures.
 298:       // The replacement should generate the same PTX as std::clamp. See https://godbolt.org/z/Wde9KW3v4
 299:       qvalue = (qvalue < qmin) ? qmin : (qmax < qvalue) ? qmax : qvalue;
 300:       *(scalar_t*)(out_data + offset) = static_cast<scalar_t>(qvalue);
 301:     }, false);
 302:   });
 303: }
```
- EN: This block defines or continues the implementation of `index_put_kernel_quantized_cuda`, `gpu_index_kernel`.
- CN: 该代码块定义或继续实现 `index_put_kernel_quantized_cuda`, `gpu_index_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 305-326
```cpp
 305: template <typename scalar_t, typename index_t, typename func_t>
 306: void cuda_take_put_kernel(
 307:   TensorIterator& iter,
 308:   const TensorBase& indexed,
 309:   const func_t& f) {
 310:   if (!iter.can_use_32bit_indexing()) {
 311:     for (auto& sub_iter : iter.with_32bit_indexing()) {
 312:       cuda_take_put_kernel<scalar_t, index_t>(sub_iter, indexed, f);
 313:     }
 314:     return;
 315:   }
 316: 
 317:   const auto numel = indexed.numel();
 318:   const bool is_contiguous = indexed.is_contiguous();
 319: 
 320:   char* const __restrict__ iterated_ptr = reinterpret_cast<char*>(iter.data_ptr(0));
 321:   char* const __restrict__ idx_ptr = reinterpret_cast<char*>(iter.data_ptr(1));
 322: 
 323:   const auto offset_calc = make_offset_calculator<2>(iter);
 324:   using uindex_t = std::make_unsigned_t<index_t>;
 325: 
 326:   // OffsetCalculator needs the sizes and strides reversed
```
- EN: This block defines or continues the implementation of `cuda_take_put_kernel`.
- CN: 该代码块定义或继续实现 `cuda_take_put_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 327-332
```cpp
 327:   const auto indexed_sizes = std::vector<int64_t>(indexed.sizes().rbegin(), indexed.sizes().rend());
 328:   const auto indexed_strides = std::vector<int64_t>(indexed.strides().rbegin(), indexed.strides().rend());
 329:   const auto* indexed_strides_data = indexed_strides.data();
 330:   const auto offset_indexed = OffsetCalculator<1, uindex_t>(indexed.dim(),
 331:                                                             indexed_sizes.data(),
 332:                                                             &indexed_strides_data);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 334-349
```cpp
 334:   const auto loop = [=]C10_DEVICE(int i) {
 335:     const auto offsets = offset_calc.get(i);
 336: 
 337:     auto& iterated = *reinterpret_cast<scalar_t*>(iterated_ptr + offsets[0]);
 338:     const auto idx = *reinterpret_cast<int64_t*>(idx_ptr + offsets[1]);
 339:     CUDA_KERNEL_ASSERT(idx < numel && idx >= -numel && "cuda_take_put_kernel() index out of bounds");
 340:     index_t offset = static_cast<index_t>(idx);
 341:     if (offset < 0) {
 342:       offset += numel;
 343:     }
 344:     if (!is_contiguous) {
 345:       offset = offset_indexed.get(offset)[0];
 346:     }
 347: 
 348:     f(iterated, offset);
 349:   };
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 350-351
```cpp
 350:   launch_kernel<launch_size_nd, launch_bound2>(iter.numel(), loop);
 351: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 353-374
```cpp
 353: void put_kernel(TensorIterator& iter, const TensorBase& output, const bool accumulate) {
 354:   AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(at::ScalarType::Half, at::ScalarType::Bool, at::ScalarType::BFloat16, iter.dtype(), "put_cuda", [&] {
 355:     // Cannot use `OpaqueType`, as we need the actual type for `fastSpecializedgpuAtomicAdd`
 356:     AT_DISPATCH_INDEX_TYPES(cuda::detail::canUse32BitIndexMath(output) ? ScalarType::Int : ScalarType::Long,
 357:         "put_cuda_index", [&] {
 358:            auto* __restrict__ indexed_ptr = output.template data_ptr<scalar_t>();
 359:            if (accumulate) {
 360:              index_t numel = output.numel();
 361:              cuda_take_put_kernel<scalar_t, index_t>(iter, output,
 362:                  [numel, indexed_ptr] __device__(scalar_t& iterated, const index_t offset) {
 363:                    fastSpecializedAtomicAdd(indexed_ptr, offset, numel, iterated);
 364:                  });
 365:            }
 366:            else {
 367:              cuda_take_put_kernel<scalar_t, index_t>(iter, output,
 368:                  [indexed_ptr] __device__(scalar_t& iterated, const index_t offset) {
 369:                    indexed_ptr[offset] = iterated;
 370:                  });
 371:            }
 372:     });
 373:   });
 374: }
```
- EN: This block defines or continues the implementation of `put_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `put_kernel`, `__device__`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 376-390
```cpp
 376: void take_kernel(
 377:   TensorIterator& iter,
 378:   const TensorBase& input) {
 379:   AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(at::ScalarType::Half, at::ScalarType::Bool, at::ScalarType::BFloat16, iter.dtype(), "take_cuda", [&] {
 380:     // Cannot use `OpaqueType`, as Tensor::data_ptr<OpaqueType<N>> is not implemented
 381:     AT_DISPATCH_INDEX_TYPES(cuda::detail::canUse32BitIndexMath(input) ? ScalarType::Int : ScalarType::Long,
 382:       "take_cuda_index", [&] {
 383:          const auto* __restrict__ indexed_ptr = input.template const_data_ptr<scalar_t>();
 384:          cuda_take_put_kernel<scalar_t, index_t>(iter, input,
 385:             [indexed_ptr] __device__(scalar_t& iterated, const index_t offset) {
 386:                iterated = indexed_ptr[offset];
 387:              });
 388:      });
 389:   });
 390: }
```
- EN: This block defines or continues the implementation of `take_kernel`, `__device__`.
- CN: 该代码块定义或继续实现 `take_kernel`, `__device__`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 392-403
```cpp
 392: namespace {
 393: 
 394: __global__ void masked_scatter_size_check(
 395:   const int64_t* const mask_exclusive_sum,
 396:   const bool* const mask,
 397:   const int64_t srcSize) {
 398:   // Convert exclusive sum to inclusive sum
 399:   const auto totalElements = *mask_exclusive_sum + *mask;
 400:   CUDA_KERNEL_ASSERT(totalElements <= srcSize);
 401: }
 402: 
 403: } // anonymous namespace
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `masked_scatter_size_check`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `masked_scatter_size_check`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 405-426
```cpp
 405: void launch_masked_scatter_kernel(
 406:     const TensorBase &self, const TensorBase &mask,
 407:     const TensorBase &maskPrefixSum, const TensorBase &source) {
 408:   const auto srcSize = source.numel();
 409:   const auto mask_cont = mask.contiguous();
 410:   const auto mask_numel = mask.numel();
 411: 
 412:   // Use a prefix sum to determine the output locations of the masked elements
 413:   auto maskPrefixSum_data = maskPrefixSum.mutable_data_ptr<int64_t>();
 414:   auto mask_data = mask_cont.const_data_ptr<bool>();
 415: 
 416:   at::cuda::cub::mask_exclusive_sum(
 417:       mask_data, maskPrefixSum_data, mask_numel);
 418: 
 419:   // Asynchronously check that the number of `1` elements present in the mask
 420:   // must be <= the number of elements available in `src`.
 421:   masked_scatter_size_check<<<1, 1, 0, at::cuda::getCurrentCUDAStream()>>>(
 422:       &maskPrefixSum_data[mask_numel - 1], &mask_data[mask_numel - 1], srcSize);
 423:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 424: 
 425:   // We are getting elements from `src` based on an offset from
 426:   // `maskPrefixSum`, so that should be made contiguous too
```
- EN: This block defines or continues the implementation of `launch_masked_scatter_kernel`.
- CN: 该代码块定义或继续实现 `launch_masked_scatter_kernel`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 427-427
```cpp
 427:   auto source_contig = source.contiguous();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 429-437
```cpp
 429:   auto iter = TensorIteratorConfig()
 430:       .set_check_mem_overlap(false)
 431:       .check_all_same_dtype(false)
 432:       .resize_outputs(false)
 433:       .add_output(self)
 434:       .add_input(self)
 435:       .add_const_input(mask_cont)
 436:       .add_input(maskPrefixSum)
 437:       .build();
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 439-455
```cpp
 439:   AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(
 440:       ScalarType::Bool,
 441:       ScalarType::BFloat16,
 442:       ScalarType::Half,
 443:       self.scalar_type(),
 444:       "masked_scatter_",
 445:       [&]() {
 446:         auto source_ptr = source_contig.const_data_ptr<scalar_t>();
 447:         gpu_kernel(
 448:             iter, [=] GPU_LAMBDA(const scalar_t a, const bool mask, const int64_t maskPrefixSum) -> scalar_t {
 449:               if (mask) {
 450:                 return source_ptr[maskPrefixSum];
 451:               }
 452:               return a;
 453:             });
 454:         AT_CUDA_CHECK(cudaGetLastError());
 455:       });
```
- EN: This block defines or continues the implementation of `gpu_kernel`.
- CN: 该代码块定义或继续实现 `gpu_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 456-456
```cpp
 456: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 458-479
```cpp
 458: template <typename scalar_t>
 459: void flip_kernel_impl(TensorIterator& iter) {
 460:   if (!iter.can_use_32bit_indexing()) {
 461:     for (auto& sub_iter : iter.with_32bit_indexing()) {
 462:       flip_kernel_impl<scalar_t>(sub_iter);
 463:     }
 464:     return;
 465:   }
 466: 
 467:   char* const __restrict__ out_ptr = reinterpret_cast<char*>(iter.data_ptr(0));
 468:   const char* const __restrict__ in_ptr = reinterpret_cast<const char*>(iter.data_ptr(1));
 469: 
 470:   const auto offset_calc = make_offset_calculator<2, /*signed_strides=*/true>(iter);
 471: 
 472:   const auto loop = [=]C10_DEVICE(const int i) {
 473:     const auto offsets = offset_calc.get(i);
 474:     // offsets can be negative here, but it's fine
 475:     scalar_t* const __restrict__ out_data = reinterpret_cast<scalar_t*>(out_ptr + offsets[0]);
 476:     const scalar_t* const __restrict__ in_data = reinterpret_cast<const scalar_t*>(in_ptr + offsets[1]);
 477:     *out_data = *in_data;
 478:   };
 479:   launch_kernel<launch_size_nd, launch_bound2>(iter.numel(), loop);
```
- EN: This block defines or continues the implementation of `flip_kernel_impl`.
- CN: 该代码块定义或继续实现 `flip_kernel_impl`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 480-480
```cpp
 480: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 482-503
```cpp
 482: void flip_kernel(TensorIterator& iter, const bool quantized) {
 483:   if (quantized) {
 484:     AT_DISPATCH_QINT_AND_SUB_BYTE_TYPES(iter.dtype(), "flip_quantized_cuda",
 485:     [&] {
 486:       using dtype = OpaqueType<sizeof(scalar_t)>;
 487:       flip_kernel_impl<dtype>(iter);
 488:     });
 489:   } else {
 490:     AT_DISPATCH_V2(
 491:       iter.dtype(),
 492:       "flip_cuda",
 493:       AT_WRAP([&] {
 494:         using dtype = OpaqueType<sizeof(scalar_t)>;
 495:         flip_kernel_impl<dtype>(iter);
 496:       }),
 497:       AT_EXPAND(AT_ALL_TYPES_AND_COMPLEX),
 498:       AT_EXPAND(AT_FLOAT8_TYPES),
 499:       AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES),
 500:       kComplexHalf,
 501:       kHalf,
 502:       kBool,
 503:       kBFloat16);
```
- EN: This block defines or continues the implementation of `flip_kernel`.
- CN: 该代码块定义或继续实现 `flip_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 504-505
```cpp
 504:   }
 505: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 508-514
```cpp
 508: REGISTER_DISPATCH(index_stub, &index_kernel)
 509: REGISTER_DISPATCH(index_fill_stub, &index_fill_kernel)
 510: REGISTER_DISPATCH(index_copy_stub, &index_copy_kernel)
 511: REGISTER_DISPATCH(index_put_stub, &index_put_kernel)
 512: REGISTER_DISPATCH(put_stub, &put_kernel)
 513: REGISTER_DISPATCH(take_stub, &take_kernel)
 514: REGISTER_DISPATCH(flip_stub, &flip_kernel)
```
- EN: The registration macro binds this implementation into PyTorch dispatch so higher-level operators can call the CUDA specialization.
- CN: 注册宏把该实现接入 PyTorch 分发系统，使上层算子能够调用这个 CUDA 特化版本。

### Lines 516-516
```cpp
 516: REGISTER_CUDA_DISPATCH(index_put_kernel_quantized_stub, &index_put_kernel_quantized_cuda)
```
- EN: The registration macro binds this implementation into PyTorch dispatch so higher-level operators can call the CUDA specialization.
- CN: 注册宏把该实现接入 PyTorch 分发系统，使上层算子能够调用这个 CUDA 特化版本。

### Lines 518-518
```cpp
 518: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TensorIterator` normalizes tensor shapes/strides and drives elementwise CUDA traversal. / `TensorIterator` 统一张量形状与步长，并驱动逐元素 CUDA 遍历。
- `TensorIteratorConfig` builds iterator state before launch. / `TensorIteratorConfig` 在启动前构建迭代器状态。
- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `gpu_kernel` applies a functor over iterator elements on CUDA. / `gpu_kernel` 在 CUDA 上对迭代器元素应用函数对象。
- Dispatch registration exposes the CUDA specialization to higher PyTorch layers. / 分发注册把 CUDA 特化实现暴露给更高层的 PyTorch 调用链。
- CUDA dispatch registration binds a stub to this file's implementation. / CUDA 分发注册会把 stub 绑定到本文件实现。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/native/cuda/IndexKernel.h>`
  - `<ATen/native/IndexKernel.h>`
  - `<array>`
  - `<type_traits>`
  - `<ATen/core/TensorBase.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/Dispatch_v2.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/cub.h>`
  - `<ATen/cuda/detail/IndexUtils.cuh>`
  - `<ATen/cuda/detail/OffsetCalculator.cuh>`
  - `<ATen/native/cuda/Loops.cuh>`
- Runtime symbols / 运行时符号:
  - `index_stub`
  - `index_fill_stub`
  - `index_copy_stub`
  - `index_put_stub`
  - `put_stub`
  - `take_stub`
  - `flip_stub`
  - `index_put_kernel_quantized_stub`
  - `launch_bound2`
  - `launch_size_nd`
  - `launch_kernel`
  - `launch_masked_scatter_kernel`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
