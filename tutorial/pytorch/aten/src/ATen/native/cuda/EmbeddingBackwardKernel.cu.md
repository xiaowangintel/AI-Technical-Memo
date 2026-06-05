# EmbeddingBackwardKernel.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/EmbeddingBackwardKernel.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `ceil_div`, `krn_partials_per_segment`, `krn_partial_segment_offset`, `compute_grad_weight_bags`.
- 用途（中文）: 实现与 `ceil_div`, `krn_partials_per_segment`, `krn_partial_segment_offset`, `compute_grad_weight_bags` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/native/cuda/EmbeddingBackwardKernel.cuh>
   3: #include <ATen/cuda/Atomic.cuh>
   4: #include <ATen/cuda/CUDAContext.h>
   5: #include <ATen/cuda/cub.cuh>
   6: #include <ATen/AccumulateType.h>
   7: #include <ATen/OpMathType.h>
   8: #include <ATen/Dispatch.h>
   9: #include <ATen/TensorUtils.h>
  10: #include <ATen/native/cuda/SortingCommon.cuh>
  11: 
  12: #include <c10/macros/Macros.h>
  13: 
  14: #include <thrust/iterator/counting_iterator.h>
  15: 
  16: #ifndef AT_PER_OPERATOR_HEADERS
  17: #include <ATen/Functions.h>
  18: #else
  19: #include <ATen/ops/empty.h>
  20: #include <ATen/ops/zeros.h>
  21: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/EmbeddingBackwardKernel.cuh>`, `<ATen/cuda/Atomic.cuh>`, `<ATen/cuda/CUDAContext.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/EmbeddingBackwardKernel.cuh>`, `<ATen/cuda/Atomic.cuh>`, `<ATen/cuda/CUDAContext.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 23-44
```cpp
  23: namespace at::native {
  24: 
  25: namespace {
  26: 
  27: /* This code computes the sum of the weights in two-steps:
  28:   1) Each GPU warp sums `NROWS_PER_THREAD` number of row given by `indices`
  29:   2) Each partial-sum from 1) are summed and scatter into `grad_weight`
  30: 
  31:   Notice, `NROWS_PER_THREAD` impacts the Achieved Occupancy of the
  32:   kernel execution. If it is high, the size of the thread blocks will be
  33:   too small to achieve good occupancy. Similarly, a very low value will
  34:   make the size of the thread blocks in the final sum in step 2) too small.
  35: */
  36: constexpr int NROWS_PER_THREAD = 10;
  37: 
  38: // If the number of blocks processed by each SM is larger than this value,
  39: // we will use the two-step sum and scatter approach.
  40: constexpr int32_t MAX_ATOMIC_ACCUM_BLOCKS_PER_SM = 4;
  41: 
  42: // Fast ceil division (no overflow checking)
  43: __host__ __device__ __forceinline__
  44: int64_t ceil_div(int64_t x, int64_t y) {
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `division`.
- CN: 该代码块定义或继续实现 `division`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 45-46
```cpp
  45:   return (x + y - 1) / y;
  46: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 48-60
```cpp
  48: template <typename index_t>
  49: __global__
  50: void krn_partials_per_segment(index_t *ret, const index_t *segment_offsets,
  51:                               const int64_t *num_of_segments_ptr, int64_t numel) {
  52:   int64_t num_of_segments = *num_of_segments_ptr;
  53:   const int id = blockIdx.x * blockDim.x + threadIdx.x;
  54:   if(id < num_of_segments) {
  55:     const int64_t idx_start = segment_offsets[id];
  56:     const int64_t idx_end = (id == num_of_segments-1)?numel:segment_offsets[id+1];
  57:     const int64_t size = idx_end - idx_start;
  58:     ret[id] = ceil_div(size, NROWS_PER_THREAD);
  59:   }
  60: }
```
- EN: This block defines GPU kernel entry point(s) `krn_partials_per_segment`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `krn_partials_per_segment`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 62-80
```cpp
  62: template <typename index_t>
  63: __global__
  64: void krn_partial_segment_offset(
  65:         index_t *ret,
  66:         const index_t *partials_per_segment,
  67:         const index_t *partials_per_segment_offset,
  68:         const index_t *segment_offsets,
  69:         const int64_t *num_of_segments_ptr) {
  70:   int64_t num_of_segments = *num_of_segments_ptr;
  71:   const int id = blockIdx.x * blockDim.x + threadIdx.x;
  72:   if(id < num_of_segments) {
  73:     index_t idx = partials_per_segment_offset[id];
  74:     const index_t num_partials = partials_per_segment[id];
  75:     const index_t segment_offset = segment_offsets[id];
  76:     for (int64_t i=0; i<num_partials; ++i) {
  77:       ret[idx++] = segment_offset + i * NROWS_PER_THREAD;
  78:     }
  79:   }
  80: }
```
- EN: This block defines GPU kernel entry point(s) `krn_partial_segment_offset`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `krn_partial_segment_offset`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 83-104
```cpp
  83: template <typename scalar_t, typename index_t>
  84: __global__ void compute_grad_weight_bags(
  85:     const index_t *indices, const scalar_t *gradOutput,
  86:     const index_t *offset2bag, const index_t *count, ptrdiff_t numel,
  87:     int64_t stride, int mode_mean, const index_t *bag_size,
  88:     const scalar_t* per_sample_weights, int64_t per_sample_weights_stride,
  89:     const index_t* segment_offsets, const int64_t *num_of_segments_ptr,
  90:     acc_type<scalar_t, true> *grad_weight_per_segment,
  91:     const int64_t stride_warped) {
  92: 
  93:   int64_t num_of_segments = *num_of_segments_ptr;
  94:   const int64_t gid = static_cast<int64_t>(blockIdx.x) * blockDim.x + threadIdx.x;
  95:   const int64_t id = gid / stride_warped;
  96:   const int64_t startFeature = gid % stride_warped;
  97:   if (startFeature >= stride) {
  98:     return;
  99:   }
 100:   if (id >= num_of_segments) {
 101:     return;
 102:   }
 103:   const int idx_begin = segment_offsets[id];
 104:   const int idx_end = (id == num_of_segments-1)?numel:segment_offsets[id+1];
```
- EN: This block defines GPU kernel entry point(s) `compute_grad_weight_bags`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `compute_grad_weight_bags`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 106-122
```cpp
 106:   acc_type<scalar_t, true> weight = 0;
 107:   for (int idx=idx_begin; idx < idx_end; ++idx) {
 108:     const int origRow = indices[idx];
 109:     const int seq_number = offset2bag[origRow];
 110:     const int gradOutputRow = seq_number * stride;
 111: 
 112:     acc_type<scalar_t, true> scale = count ? 1.0 / count[idx] : 1.0;
 113:     if (per_sample_weights) {
 114:       scale *= per_sample_weights[origRow * per_sample_weights_stride];
 115:     }
 116: 
 117:     acc_type<scalar_t, true> gradient = gradOutput[gradOutputRow + startFeature];
 118:     if (mode_mean) {
 119:       gradient /= bag_size[seq_number];
 120:     }
 121:     weight += gradient * scale;
 122:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 123-124
```cpp
 123:   grad_weight_per_segment[id * stride + startFeature] = weight;
 124: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 126-147
```cpp
 126: template <typename scalar_t, typename index_t>
 127: __global__ void compute_grad_weight(
 128:     const index_t *indices,
 129:     const scalar_t *gradOutput,
 130:     const index_t *count,
 131:     ptrdiff_t numel,
 132:     int64_t stride,
 133:     const index_t* segment_offsets,
 134:     const int64_t *num_of_segments_ptr,
 135:     acc_type<scalar_t, true> *grad_weight_per_segment,
 136:     const int64_t stride_warped) {
 137: 
 138:   int64_t num_of_segments = *num_of_segments_ptr;
 139:   using accscalar_t = acc_type<scalar_t, true>;
 140:   const int64_t gid = static_cast<int64_t>(blockIdx.x) * blockDim.x + threadIdx.x;
 141:   const int64_t id = gid / stride_warped;
 142:   const int64_t startFeature = gid % stride_warped;
 143:   if (startFeature >= stride) {
 144:     return;
 145:   }
 146:   if (id >= num_of_segments) {
 147:     return;
```
- EN: This block defines GPU kernel entry point(s) `compute_grad_weight`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `compute_grad_weight`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 148-150
```cpp
 148:   }
 149:   const int idx_begin = segment_offsets[id];
 150:   const int idx_end = (id == num_of_segments-1)?numel:segment_offsets[id+1];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 152-159
```cpp
 152:   accscalar_t weight = 0;
 153:   for (int idx=idx_begin; idx < idx_end; ++idx) {
 154:     const index_t target_row = indices[idx];
 155:     const accscalar_t scale = count ? (accscalar_t)1.0 / count[idx] : 1.0;
 156:     weight += gradOutput[target_row * stride + startFeature] * scale;
 157:   }
 158:   grad_weight_per_segment[id * stride + startFeature] = weight;
 159: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 161-166
```cpp
 161: // Fused kernel that combines compute_grad_weight and sum_and_scatter using atomic adds.
 162: // This eliminates the serialization bottleneck when num_of_segments is small.
 163: // Each partial segment atomically adds its contribution directly to grad_weight.
 164: // Template parameters:
 165: //   scalar_t: input gradient type
 166: //   output_t: output gradient weight type (float for half/bf16 atomic accumulation)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 167-180
```cpp
 167: template <typename scalar_t, typename output_t, typename index_t>
 168: __global__ void compute_grad_weight_atomic_accumulate(
 169:     const index_t *orig_indices,
 170:     const scalar_t *gradOutput,
 171:     const index_t *count,
 172:     ptrdiff_t numel,
 173:     int64_t stride,
 174:     const index_t *partial_segment_offsets,
 175:     const int64_t *num_of_partial_segments_ptr,
 176:     const index_t *sorted_indices,
 177:     const index_t *partial_to_segment_idx,
 178:     const index_t *segment_offsets,
 179:     output_t *grad_weight,
 180:     const int64_t padding_idx,
```
- EN: This block defines GPU kernel entry point(s) `compute_grad_weight_atomic_accumulate`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `compute_grad_weight_atomic_accumulate`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 181-202
```cpp
 181:     const int64_t stride_warped) {
 182: 
 183:   int64_t num_of_partial_segments = *num_of_partial_segments_ptr;
 184:   using accscalar_t = acc_type<scalar_t, true>;
 185:   const int32_t gid = blockIdx.x * blockDim.x + threadIdx.x;
 186:   const int32_t partial_id = gid / stride_warped;
 187:   const int32_t startFeature = gid % stride_warped;
 188: 
 189:   if (startFeature >= stride) {
 190:     return;
 191:   }
 192:   if (partial_id >= num_of_partial_segments) {
 193:     return;
 194:   }
 195: 
 196:   const index_t idx_begin = partial_segment_offsets[partial_id];
 197:   const index_t idx_end = (partial_id == num_of_partial_segments - 1)
 198:       ? numel
 199:       : partial_segment_offsets[partial_id + 1];
 200: 
 201:   accscalar_t weight = 0;
 202:   for (index_t idx = idx_begin; idx < idx_end; ++idx) {
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 203-206
```cpp
 203:     const index_t target_row = orig_indices[idx];
 204:     const accscalar_t scale = count ? (accscalar_t)1.0 / count[idx] : 1.0;
 205:     weight += gradOutput[target_row * stride + startFeature] * scale;
 206:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 208-208
```cpp
 208:   // Get the target row for this partial segment from the segment info
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 209-210
```cpp
 209:   const index_t segment_id = partial_to_segment_idx[partial_id];
 210:   const index_t target_row = sorted_indices[segment_offsets[segment_id]];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 212-217
```cpp
 212:   if (target_row != padding_idx) {
 213:     gpuAtomicAddNoReturn(
 214:         &grad_weight[target_row * stride + startFeature],
 215:         static_cast<output_t>(weight));
 216:   }
 217: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 219-219
```cpp
 219: // Kernel to build mapping from partial segment ID to segment ID
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 220-235
```cpp
 220: template <typename index_t>
 221: __global__ void krn_partial_to_segment_idx(
 222:     index_t *partial_to_segment_idx,
 223:     const index_t *partials_per_segment,
 224:     const index_t *partials_per_segment_offset,
 225:     const int64_t *num_of_segments_ptr) {
 226:   int64_t num_of_segments = *num_of_segments_ptr;
 227:   const int32_t id = blockIdx.x * blockDim.x + threadIdx.x;
 228:   if (id < num_of_segments) {
 229:     index_t idx = partials_per_segment_offset[id];
 230:     const index_t num_partials = partials_per_segment[id];
 231:     for (index_t i = 0; i < num_partials; ++i) {
 232:       partial_to_segment_idx[idx++] = id;
 233:     }
 234:   }
 235: }
```
- EN: This block defines GPU kernel entry point(s) `krn_partial_to_segment_idx`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `krn_partial_to_segment_idx`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 237-237
```cpp
 237: // This kernel assumes that all input tensors are contiguous.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 238-259
```cpp
 238: template <typename scalar_t, typename index_t>
 239: __global__ void sum_and_scatter(
 240:     const index_t *input, scalar_t *gradWeight, int64_t stride,
 241:     const index_t* segment_offsets, const int64_t *num_of_segments_ptr,
 242:     const acc_type<scalar_t, true> *grad_weight_per_segment,
 243:     const index_t *segment_sizes_offsets, const int64_t *num_of_partial_segments_ptr,
 244:     const int64_t padding_idx,
 245:     const int64_t stride_warped) {
 246: 
 247:   int64_t num_of_segments = *num_of_segments_ptr;
 248:   int64_t num_of_partial_segments = *num_of_partial_segments_ptr;
 249:   const int64_t gid = static_cast<int64_t>(blockIdx.x) * blockDim.x + threadIdx.x;
 250:   const int64_t id = gid / stride_warped;
 251:   const int64_t startFeature = gid % stride_warped;
 252:   if (startFeature >= stride) {
 253:     return;
 254:   }
 255:   if (id >= num_of_segments) {
 256:     return;
 257:   }
 258: 
 259:   const int idx_begin = segment_sizes_offsets[id];
```
- EN: This block defines GPU kernel entry point(s) `sum_and_scatter`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `sum_and_scatter`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 260-269
```cpp
 260:   const int idx_end = (id == num_of_segments-1)?num_of_partial_segments:segment_sizes_offsets[id+1];
 261:   acc_type<scalar_t, true> weight = 0;
 262:   for (int idx=idx_begin; idx < idx_end; ++idx) {
 263:     weight += grad_weight_per_segment[idx*stride + startFeature];
 264:   }
 265:   int64_t target_row = input[segment_offsets[id]];
 266:   if (target_row != padding_idx) {
 267:     gradWeight[target_row * stride + startFeature] = weight;
 268:   }
 269: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 271-276
```cpp
 271: template<typename index_t>
 272: __global__ void compute_num_of_partial_segments(const index_t *partials_per_segment, const index_t *partials_per_segment_offset, const int64_t *num_of_segments_ptr, int64_t *output) {
 273:   int64_t num_of_segments = *num_of_segments_ptr;
 274:   *output = partials_per_segment[num_of_segments-1] +
 275:             partials_per_segment_offset[num_of_segments-1];
 276: }
```
- EN: This block defines GPU kernel entry point(s) `compute_num_of_partial_segments`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `compute_num_of_partial_segments`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 279-279
```cpp
 279: } // anon namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 282-303
```cpp
 282: Tensor embedding_backward_cuda_kernel(
 283:         const Tensor &grad,
 284:         const Tensor &orig_indices,
 285:         const Tensor &sorted_indices,
 286:         const Tensor &count,
 287:         int64_t num_weights,
 288:         int padding_idx,
 289:         bool mode_mean,
 290:         const Tensor &offset2bag,
 291:         const Tensor &bag_size,
 292:         const Tensor &per_sample_weights) {
 293: 
 294:   auto stream = at::cuda::getCurrentCUDAStream();
 295:   const ptrdiff_t numel = sorted_indices.numel();
 296: 
 297:   auto grad_weight = at::zeros({num_weights, grad.size(-1)}, grad.options());
 298:   const int64_t stride = grad_weight.stride(0);
 299: 
 300:   // Compute the number of segments and their start position so that we do not have to
 301:   // spawn a warp per index. In this context, a segment is a number of rows that should
 302:   // be summarized.
 303:   // Unit: index in `sorted_indices` and `orig_indices`
```
- EN: This block defines or continues the implementation of `embedding_backward_cuda_kernel`.
- CN: 该代码块定义或继续实现 `embedding_backward_cuda_kernel`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 304-312
```cpp
 304:   auto segment_offsets = at::empty({numel}, orig_indices.options());
 305:   auto num_of_segments_tensor = at::empty({}, grad.options().dtype(kLong));
 306:   int64_t *num_of_segments_ptr = num_of_segments_tensor.mutable_data_ptr<int64_t>();
 307:   AT_DISPATCH_INDEX_TYPES(orig_indices.scalar_type(), "embedding_backward_cuda_kernel", [&] () {
 308:     cuda::cub::unique_by_key(
 309:       sorted_indices.const_data_ptr<index_t>(), thrust::make_counting_iterator(0),
 310:       segment_offsets.mutable_data_ptr<index_t>(),
 311:       num_of_segments_ptr, sorted_indices.numel());
 312:   });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 314-314
```cpp
 314:   int64_t max_segments = std::min<int64_t>(numel, num_weights);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 316-337
```cpp
 316:   AT_DISPATCH_INDEX_TYPES(orig_indices.scalar_type(), "embedding_backward_cuda_kernel", [&] () {
 317:     // We split the segments up into sizes of `NROWS_PER_THREAD`
 318:     // Compute the number partial-segments per segment (some partial-segments
 319:     // may not be the full `NROWS_PER_THREAD` number of rows)
 320:     auto partials_per_segment = at::empty({max_segments}, orig_indices.options());
 321:     {
 322:       krn_partials_per_segment<<<ceil_div(max_segments, 32), 32, 0, stream>>> (
 323:               partials_per_segment.mutable_data_ptr<index_t>(),
 324:               segment_offsets.const_data_ptr<index_t>(),
 325:               num_of_segments_ptr,
 326:               numel);
 327:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 328:     }
 329: 
 330:     // In order to compute `partial_segment_offset`, which is the start index
 331:     // of each partial-segment in `sorted_indices`, we need to compute the
 332:     // start position of each _segment_ in `partial_segment_offset`.
 333:     // Unit: index in `partial_segment_offset`
 334:     auto partials_per_segment_offset = at::empty({max_segments}, orig_indices.options());
 335:     cuda::cub::exclusive_sum(
 336:         partials_per_segment.const_data_ptr<index_t>(),
 337:         partials_per_segment_offset.mutable_data_ptr<index_t>(),
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 338-338
```cpp
 338:         max_segments);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 340-340
```cpp
 340:     // The total number of partial-segments is the sum of `partials_per_segment_offset`
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 341-347
```cpp
 341:     auto num_of_partial_segments_tensor = at::empty({}, grad.options().dtype(kLong));
 342:     int64_t *num_of_partial_segments_ptr = num_of_partial_segments_tensor.mutable_data_ptr<int64_t>();
 343:     compute_num_of_partial_segments<index_t><<<1, 1, 0, c10::cuda::getCurrentCUDAStream()>>>(
 344:       partials_per_segment.const_data_ptr<index_t>(),
 345:       partials_per_segment_offset.const_data_ptr<index_t>(),
 346:       num_of_segments_ptr, num_of_partial_segments_ptr);
 347:     C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 349-349
```cpp
 349:     auto max_partial_segment = numel / NROWS_PER_THREAD + max_segments;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 351-352
```cpp
 351:     // Now we can compute the start position of each partial-segment
 352:     // Unit: index in `sorted_indices` and `orig_indices`
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 353-362
```cpp
 353:     auto partial_segment_offset = at::empty({max_partial_segment}, orig_indices.options());
 354:     {
 355:       krn_partial_segment_offset<<<ceil_div(max_segments, 32), 32, 0, stream>>> (
 356:               partial_segment_offset.mutable_data_ptr<index_t>(),
 357:               partials_per_segment.const_data_ptr<index_t>(),
 358:               partials_per_segment_offset.const_data_ptr<index_t>(),
 359:               segment_offsets.const_data_ptr<index_t>(),
 360:               num_of_segments_ptr);
 361:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 362:     }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 364-367
```cpp
 364:     const int warp_size = at::cuda::warp_size();
 365:     const int stride_warped = ceil_div(stride, warp_size)*warp_size;
 366:     const int block = std::min(stride_warped, MAX_BLOCK_SIZE);
 367:     const int grid = ceil_div(max_partial_segment*stride_warped, block);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 369-374
```cpp
 369:     // Heuristic: Use fused kernel when sum_and_scatter would have poor parallelism
 370:     // This happens when max_segments * stride_warped is small relative to GPU capacity
 371:     // The fused kernel uses atomic adds but has grid size proportional to max_partial_segment
 372:     // instead of max_segments, giving much better parallelism when segments are few but large
 373:     // Note: Fused kernel uses atomics which are non-deterministic, so we skip it
 374:     // when deterministic algorithms are requested
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 375-381
```cpp
 375:     const int32_t sum_scatter_grid = ceil_div(max_segments * stride_warped, block);
 376:     const int32_t num_sms = at::cuda::getCurrentDeviceProperties()->multiProcessorCount;
 377:     const int32_t min_blocks_for_good_occupancy = num_sms * MAX_ATOMIC_ACCUM_BLOCKS_PER_SM;
 378:     const bool use_fused_kernel = !offset2bag.defined() &&
 379:         sum_scatter_grid < min_blocks_for_good_occupancy &&
 380:         max_partial_segment > sum_scatter_grid * 4 &&
 381:         !at::globalContext().deterministicAlgorithms();
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 383-404
```cpp
 383:     AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16,
 384:       grad.scalar_type(), "embedding_bag_backward_cuda_compute_grad_weight", [&] {
 385:         // For numerical stability, the dtype of `grad_weight_per_segment`
 386:         // should match `acc_type`
 387:         using partial_weight_t = acc_type<scalar_t, true>;
 388: 
 389:         if (use_fused_kernel) {
 390:           // This eliminates the serialization bottleneck in sum_and_scatter
 391:           // by directly accumulating into grad_weight from all partial segments
 392: 
 393:           // Build mapping from partial segment ID to segment ID
 394:           auto partial_to_segment_idx = at::empty({max_partial_segment}, orig_indices.options());
 395:           {
 396:             krn_partial_to_segment_idx<<<ceil_div(max_segments, 32), 32, 0, stream>>>(
 397:                 partial_to_segment_idx.mutable_data_ptr<index_t>(),
 398:                 partials_per_segment.const_data_ptr<index_t>(),
 399:                 partials_per_segment_offset.const_data_ptr<index_t>(),
 400:                 num_of_segments_ptr);
 401:             C10_CUDA_KERNEL_LAUNCH_CHECK();
 402:           }
 403: 
 404:           // For half/bfloat16 types, use opmath_type (float) intermediate buffer for atomic adds
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 405-406
```cpp
 405:           // (float atomics are faster and more precise than half/bf16 atomics)
 406:           // For float/double, accumulate directly into grad_weight
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 407-428
```cpp
 407:           if constexpr (!std::is_same_v<scalar_t, opmath_type<scalar_t>>) {
 408:             auto grad_weight_acc = at::zeros(
 409:                 {num_weights, stride},
 410:                 grad.options().dtype(toOpMathType(grad.scalar_type())));
 411:             compute_grad_weight_atomic_accumulate<scalar_t, partial_weight_t><<<grid, block, 0, stream>>>(
 412:                 orig_indices.const_data_ptr<index_t>(),
 413:                 grad.const_data_ptr<scalar_t>(),
 414:                 count.defined() ? count.const_data_ptr<index_t>() : nullptr,
 415:                 numel, stride,
 416:                 partial_segment_offset.const_data_ptr<index_t>(),
 417:                 num_of_partial_segments_ptr,
 418:                 sorted_indices.const_data_ptr<index_t>(),
 419:                 partial_to_segment_idx.const_data_ptr<index_t>(),
 420:                 segment_offsets.const_data_ptr<index_t>(),
 421:                 grad_weight_acc.mutable_data_ptr<partial_weight_t>(),
 422:                 padding_idx,
 423:                 stride_warped);
 424:             C10_CUDA_KERNEL_LAUNCH_CHECK();
 425:             // Convert back to original dtype
 426:             grad_weight.copy_(grad_weight_acc);
 427:           } else {
 428:             // For float/double, accumulate directly into grad_weight
```
- EN: This block defines or continues the implementation of `constexpr`.
- CN: 该代码块定义或继续实现 `constexpr`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 429-442
```cpp
 429:             compute_grad_weight_atomic_accumulate<scalar_t, scalar_t><<<grid, block, 0, stream>>>(
 430:                 orig_indices.const_data_ptr<index_t>(),
 431:                 grad.const_data_ptr<scalar_t>(),
 432:                 count.defined() ? count.const_data_ptr<index_t>() : nullptr,
 433:                 numel, stride,
 434:                 partial_segment_offset.const_data_ptr<index_t>(),
 435:                 num_of_partial_segments_ptr,
 436:                 sorted_indices.const_data_ptr<index_t>(),
 437:                 partial_to_segment_idx.const_data_ptr<index_t>(),
 438:                 segment_offsets.const_data_ptr<index_t>(),
 439:                 grad_weight.mutable_data_ptr<scalar_t>(),
 440:                 padding_idx,
 441:                 stride_warped);
 442:             C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 443-444
```cpp
 443:           }
 444:         } else {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 446-446
```cpp
 446:           // Two-pass path: compute_grad_weight + sum_and_scatter
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 447-468
```cpp
 447:           auto grad_weight_per_segment = at::empty(
 448:               {max_partial_segment, stride},
 449:               grad.options().dtype(toOpMathType(grad.scalar_type())));
 450:           // Compute the sum of each partial-segment and handle bags
 451:           if (offset2bag.defined()) {
 452:                 compute_grad_weight_bags<scalar_t><<<grid, block, 0, stream>>>(
 453:                   orig_indices.const_data_ptr<index_t>(),
 454:                   grad.const_data_ptr<scalar_t>(),
 455:                   offset2bag.const_data_ptr<index_t>(),
 456:                   count.defined() ? count.const_data_ptr<index_t>() : nullptr, numel, stride,
 457:                   mode_mean, bag_size.const_data_ptr<index_t>(),
 458:                   per_sample_weights.defined() ? per_sample_weights.const_data_ptr<scalar_t>() : NULL,
 459:                   per_sample_weights.defined() ? per_sample_weights.stride(0) : 0,
 460:                   partial_segment_offset.const_data_ptr<index_t>(),
 461:                   num_of_partial_segments_ptr, grad_weight_per_segment.mutable_data_ptr<partial_weight_t>(),
 462:                   stride_warped);
 463:                 C10_CUDA_KERNEL_LAUNCH_CHECK();
 464:           } else {
 465:                 compute_grad_weight<scalar_t><<<grid, block, 0, stream>>>(
 466:                   orig_indices.const_data_ptr<index_t>(),
 467:                   grad.const_data_ptr<scalar_t>(),
 468:                   count.defined() ? count.const_data_ptr<index_t>() : nullptr,
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 469-475
```cpp
 469:                   numel, stride,
 470:                   partial_segment_offset.const_data_ptr<index_t>(),
 471:                   num_of_partial_segments_ptr,
 472:                   grad_weight_per_segment.mutable_data_ptr<partial_weight_t>(),
 473:                   stride_warped);
 474:                 C10_CUDA_KERNEL_LAUNCH_CHECK();
 475:           }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 477-478
```cpp
 477:           // Finally, we sum all the partial-sums and scatter them
 478:           // into `grad_weight`.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 479-492
```cpp
 479:           const int grid2 = ceil_div(max_segments*stride_warped, block);
 480:               sum_and_scatter<scalar_t><<<grid2, block, 0, stream>>>(
 481:                 sorted_indices.const_data_ptr<index_t>(),
 482:                 grad_weight.mutable_data_ptr<scalar_t>(),
 483:                 stride,
 484:                 segment_offsets.const_data_ptr<index_t>(),
 485:                 num_of_segments_ptr, grad_weight_per_segment.const_data_ptr<partial_weight_t>(),
 486:                 partials_per_segment_offset.const_data_ptr<index_t>(),
 487:                 num_of_partial_segments_ptr,
 488:                 padding_idx,
 489:                 stride_warped);
 490:               C10_CUDA_KERNEL_LAUNCH_CHECK();
 491:         }
 492:     });
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 493-495
```cpp
 493:   });
 494:   return grad_weight;
 495: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 497-497
```cpp
 497: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- CUB primitives provide parallel CUDA building blocks such as reductions or scans. / CUB 原语提供归约、扫描等并行 CUDA 基础组件。
- Thrust utilities supply STL-like CUDA algorithms and containers. / Thrust 工具提供类似 STL 的 CUDA 算法与容器。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/native/cuda/EmbeddingBackwardKernel.cuh>`
  - `<ATen/cuda/Atomic.cuh>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/cub.cuh>`
  - `<ATen/AccumulateType.h>`
  - `<ATen/OpMathType.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/TensorUtils.h>`
  - `<ATen/native/cuda/SortingCommon.cuh>`
  - `<c10/macros/Macros.h>`
  - `<thrust/iterator/counting_iterator.h>`
  - `<ATen/Functions.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_INDEX_TYPES`
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `cub::unique_by_key`
  - `cub::exclusive_sum`
  - `thrust::make_counting_iterator`
  - `at::cuda::getCurrentCUDAStream`
  - `at::cuda::warp_size`
  - `at::cuda::getCurrentDeviceProperties`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
