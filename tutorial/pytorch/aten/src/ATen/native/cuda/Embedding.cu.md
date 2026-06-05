# Embedding.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/Embedding.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `embedding_backward_kernel`, `renorm_kernel`, `embedding_dense_backward_cuda`, `embedding_renorm_cuda_`.
- 用途（中文）: 实现与 `embedding_backward_kernel`, `renorm_kernel`, `embedding_dense_backward_cuda`, `embedding_renorm_cuda_` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/AccumulateType.h>
   4: #include <ATen/Dispatch.h>
   5: #include <ATen/TensorUtils.h>
   6: #include <ATen/ceil_div.h>
   7: #include <ATen/cuda/CUDAContext.h>
   8: #include <c10/util/Exception.h>
   9: #include <c10/macros/Macros.h>
  10: 
  11: #include <ATen/cuda/cub.cuh>
  12: 
  13: #include <ATen/native/cuda/EmbeddingBackwardKernel.cuh>
  14: #include <ATen/native/cuda/SortingCommon.cuh>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/Dispatch.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/Dispatch.h>`。

### Lines 15-28
```cpp
  15: #include <ATen/native/cuda/block_reduce.cuh>
  16: #include <ATen/native/cuda/thread_constants.h>
  17: 
  18: #include <thrust/iterator/reverse_iterator.h>
  19: 
  20: #ifndef AT_PER_OPERATOR_HEADERS
  21: #include <ATen/Functions.h>
  22: #include <ATen/NativeFunctions.h>
  23: #else
  24: #include <ATen/ops/arange.h>
  25: #include <ATen/ops/embedding_dense_backward_native.h>
  26: #include <ATen/ops/embedding_renorm_native.h>
  27: #include <ATen/ops/empty.h>
  28: #include <ATen/ops/empty_like.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/block_reduce.cuh>`, `<ATen/native/cuda/thread_constants.h>`, `<thrust/iterator/reverse_iterator.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/block_reduce.cuh>`, `<ATen/native/cuda/thread_constants.h>`, `<thrust/iterator/reverse_iterator.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 29-30
```cpp
  29: #include <ATen/ops/zeros.h>
  30: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/ops/zeros.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/ops/zeros.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 32-53
```cpp
  32: namespace at::native {
  33: 
  34: namespace {
  35: 
  36: #if defined(USE_ROCM)
  37: static constexpr int BLOCKDIMY = 16;
  38: #else
  39: static constexpr int BLOCKDIMY = 32;
  40: #endif
  41: 
  42: template
  43:   <typename scalar_t,
  44:    typename accscalar_t,
  45:    typename index_t>
  46: __global__ void embedding_backward_feature_kernel
  47:   (const index_t* indices,
  48:    const scalar_t* __restrict__ grad,
  49:    scalar_t* __restrict__ grad_weight,
  50:    int n, // OK to pass as int, we don't expect 2 billion+ samples in one shot
  51:    int64_t stride,
  52:    int padding_idx)
  53: {
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `embedding_backward_feature_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `embedding_backward_feature_kernel`，它们会直接在 CUDA 线程上执行。

### Lines 54-57
```cpp
  54:   extern __shared__ char buf[];
  55:   accscalar_t* smem = (accscalar_t*)buf;
  56:   accscalar_t* my_s = smem + C10_WARP_SIZE*threadIdx.y;
  57:   int* indices_batch = (int*)(buf + sizeof(accscalar_t)*C10_WARP_SIZE*blockDim.y);
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 59-59
```cpp
  59:   const int s = (int)stride; // OK to make int, we don't expect 2 billion+ embedding row size
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 61-61
```cpp
  61:   const int f = threadIdx.x + blockIdx.x*blockDim.x; // feature_dim
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 63-84
```cpp
  63:   for(int batch_start = 0; batch_start < n; batch_start += blockDim.x*blockDim.y)
  64:   {
  65:     // Entire block cooperates to load a batch of 1024 indices to process
  66:     int tid = threadIdx.x + threadIdx.y*blockDim.x;
  67:     if(batch_start + tid < n)
  68:       indices_batch[tid] = (int)indices[batch_start + tid];
  69: 
  70:     int batch_end = batch_start + blockDim.x*blockDim.y < n ?
  71:                     batch_start + blockDim.x*blockDim.y : n;
  72: 
  73:     // Loop over the batch of <= 1024 loaded indices in chunks of blockDim.y = 32
  74:     for(int chunk_start = batch_start; chunk_start < batch_end; chunk_start += blockDim.y)
  75:     {
  76:       // This does double duty:  it makes sure indices_batch is ready, and it makes sure match-group
  77:       // leaders are done with their accumulates before other warps start loading again.
  78:       __syncthreads();
  79: 
  80:       int n_this_chunk = (batch_end - chunk_start) < blockDim.y ?
  81:                          (batch_end - chunk_start) : blockDim.y;
  82: 
  83:       int src_row = chunk_start + threadIdx.y;
  84:       int dst_row = indices_batch[src_row - batch_start]; // This warp's target row in grad_weight
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 86-86
```cpp
  86:       // All warps load their smem segments with incoming grad data
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 87-88
```cpp
  87:       if(src_row < n && f < s && dst_row != padding_idx)
  88:         my_s[threadIdx.x] = static_cast<accscalar_t>(grad[src_row*stride + f]);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 90-90
```cpp
  90:       __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 92-96
```cpp
  92:       // To ensure determinism, we can't just have each warp add its grad data to its dst_row.
  93:       // We need to check if any other warps pulled grad data targeting dst_row.
  94:       // If so, we elect the first warp in each matching group as the leader.
  95:       // Each leader warp serializes the accumulates targeting dst_row in shared memory,
  96:       // then finishes by adding the accumulated buffer to dst_row in grad_weight.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 97-118
```cpp
  97:       if(dst_row != padding_idx && src_row < n) // Per-warp exit condition, safe with ballot_sync
  98:       {
  99:         int match_found_this_thread = 0;
 100:         if(threadIdx.x < n_this_chunk)
 101:           match_found_this_thread = (dst_row == indices_batch[chunk_start - batch_start + threadIdx.x]);
 102: #if defined(USE_ROCM)
 103:         unsigned long long int matchmask = WARP_BALLOT(match_found_this_thread);
 104:         int first_remaining_peer = __ffsll(matchmask) - 1;
 105: #else
 106:         unsigned int matchmask = WARP_BALLOT(match_found_this_thread);
 107:         int first_remaining_peer = __ffs(matchmask) - 1;
 108: #endif
 109: 
 110:         if(threadIdx.y == first_remaining_peer) // Nominate lowest-indexed warp as the leader
 111:         {
 112:           matchmask ^= (1 << first_remaining_peer);
 113:           while(matchmask)
 114:           {
 115: #if defined(USE_ROCM)
 116:             first_remaining_peer = __ffsll(matchmask) - 1;
 117: #else
 118:             first_remaining_peer = __ffs(matchmask) - 1;
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 119-119
```cpp
 119: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 120-129
```cpp
 120:             my_s[threadIdx.x] += smem[threadIdx.x + C10_WARP_SIZE*first_remaining_peer];
 121:             matchmask ^= (1 << first_remaining_peer);
 122:           }
 123:           if(f < s)
 124:             grad_weight[dst_row*stride + f] += static_cast<scalar_t>(my_s[threadIdx.x]);
 125:         }
 126:       }
 127:     }
 128:   }
 129: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 132-152
```cpp
 132: template <typename scalar_t, typename index_t>
 133: __global__ void embedding_backward_kernel(
 134:   index_t* input, index_t* indices, scalar_t* grad_output, scalar_t* grad_weight,
 135:   index_t* count, int64_t numel, int64_t stride, int padding_idx) {
 136: 
 137:   using accscalar_t = acc_type<scalar_t, true>;
 138:   int idx = blockIdx.x * 4 + threadIdx.y;
 139: 
 140:   // Each warp is responsible for an input into the LookupTable.
 141:   // If the preceding input has the same as this input, then the warp
 142:   // exits immediately. The warp also processes subsequent inputs with the
 143:   // same value.
 144:   //
 145:   // Input Warp
 146:   // 1     <warp 1>
 147:   // 1     <warp 1> (<warp 2> exits without doing any work)
 148:   // 5     <warp 3>
 149:   // 8     <warp 4>
 150: 
 151:   // Number of values processed by each thread (grain size)
 152:   const int SZ = 4;
```
- EN: This block defines GPU kernel entry point(s) `embedding_backward_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `embedding_backward_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 154-175
```cpp
 154:   if (idx < numel
 155:       && (idx == 0 || input[idx] != input[idx - 1])
 156:       && input[idx] != padding_idx) {
 157:     do {
 158:       const int start_feature = threadIdx.x + blockIdx.y * blockDim.x * SZ;
 159:       const int weight_row = ((int) input[idx]) * stride;
 160:       const int grad_row = ((int) indices[idx]) * stride;
 161:       const accscalar_t scale = count ? (accscalar_t)1.0 / count[idx] : 1.0;
 162: 
 163:       accscalar_t gradient[SZ];
 164:       accscalar_t weight[SZ];
 165: 
 166:       #pragma unroll
 167:       for (int ii = 0; ii < SZ; ii++) {
 168:         int feature_dim = start_feature + ii * C10_WARP_SIZE;
 169:         if (feature_dim < stride) {
 170:           gradient[ii] = static_cast<accscalar_t>(grad_output[grad_row + feature_dim]);
 171:           weight[ii] = static_cast<accscalar_t>(grad_weight[weight_row + feature_dim]);
 172:         }
 173:       }
 174: 
 175:       #pragma unroll
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 176-178
```cpp
 176:       for (int ii = 0; ii < SZ; ii++) {
 177:         weight[ii] += gradient[ii] * scale;
 178:       }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 180-180
```cpp
 180:       #pragma unroll
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 181-186
```cpp
 181:       for (int ii = 0; ii < SZ; ii++) {
 182:         int feature_dim = start_feature + ii * C10_WARP_SIZE;
 183:         if (feature_dim < stride) {
 184:             grad_weight[weight_row + feature_dim] = static_cast<scalar_t>(weight[ii]);
 185:         }
 186:       }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 188-191
```cpp
 188:       idx++;
 189:     } while (idx < numel && input[idx] == input[idx - 1]);
 190:   }
 191: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 193-193
```cpp
 193: /* Calculate norms of the rows of weight_ptr given by idx_ptr and capture them in norms */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 194-215
```cpp
 194: template <typename scalar_t, typename accscalar_t, typename index_t>
 195: __global__ void renorm_kernel(
 196:     scalar_t* weights, index_t* indices, accscalar_t max_norm,
 197:     accscalar_t norm_type, int64_t dim,
 198:     int64_t weights_stride0, int64_t weights_stride1,
 199:     const int64_t *num_unique_indices) {
 200:   if (blockIdx.x >= *num_unique_indices) {
 201:     return;
 202:   }
 203: 
 204:   // Some casting hacks since dynamic shared memory and templates don't work together:
 205:   extern __shared__ unsigned char smem[];
 206:   auto sdata = reinterpret_cast<accscalar_t*>(smem);
 207: 
 208:   int tid = threadIdx.x;
 209:   int base_index = indices[blockIdx.x] * weights_stride0;
 210: 
 211:   accscalar_t v = 0;
 212:   for (int i = tid; i < dim; i += blockDim.x) {
 213:     auto x = static_cast<accscalar_t>(weights[base_index + i * weights_stride1]);
 214:     if (norm_type == 1) {
 215:       v += std::abs(x);
```
- EN: This block defines GPU kernel entry point(s) `renorm_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `renorm_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 216-221
```cpp
 216:     } else if (norm_type == 2) {
 217:       v += x * x;
 218:     } else {
 219:       v += std::pow(x, norm_type);
 220:     }
 221:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 223-223
```cpp
 223:   v = cuda_utils::BlockReduceSum(v, sdata);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 225-228
```cpp
 225:   if (tid == 0) {
 226:     sdata[0] = std::pow(v, static_cast<accscalar_t>(1.0 / norm_type));
 227:   }
 228:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 230-230
```cpp
 230:   // now we renormalize the blocks that need it
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 231-237
```cpp
 231:   if (sdata[0] > max_norm) {
 232:     auto factor = static_cast<scalar_t>(max_norm / (sdata[0] + 1e-7));
 233:     for (int i = tid; i < dim; i += blockDim.x) {
 234:       weights[base_index + i * weights_stride1] *= factor;
 235:     }
 236:   }
 237: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 239-239
```cpp
 239: } // anonymous namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 242-262
```cpp
 242: Tensor embedding_dense_backward_cuda(const Tensor & grad_, const Tensor & indices_,
 243:                                int64_t num_weights, int64_t padding_idx,
 244:                                bool scale_grad_by_freq) {
 245:   auto grad_arg = TensorArg(grad_, "grad", 1);
 246:   auto indices_arg = TensorArg(indices_, "indices", 1);
 247:   checkScalarTypes("embedding_backward", indices_arg, {kLong, kInt});
 248:   checkSameGPU("embedding_backward", grad_arg, indices_arg);
 249: 
 250:   auto indices = indices_.contiguous();
 251: 
 252:   auto num_indices = indices.numel();
 253:   auto grad = grad_.contiguous().view({num_indices, grad_.size(-1)});
 254:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
 255: 
 256:   if (num_indices <= 3072 && !scale_grad_by_freq) {
 257:     auto indices_contig = indices.contiguous();
 258:     auto grad_weight = at::zeros({num_weights, grad_.size(-1)}, grad_.options());
 259:     int64_t stride = grad_weight.stride(0);
 260:     int warp_size = at::cuda::warp_size();
 261:     dim3 grid(ceil_div(stride, (int64_t)warp_size));
 262:     dim3 block(warp_size, BLOCKDIMY);
```
- EN: This block defines or continues the implementation of `embedding_dense_backward_cuda`.
- CN: 该代码块定义或继续实现 `embedding_dense_backward_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 264-285
```cpp
 264:     AT_DISPATCH_FLOATING_TYPES_AND2(
 265:       at::ScalarType::Half, at::ScalarType::BFloat16,
 266:       grad.scalar_type(),
 267:        "embedding_backward",
 268:        [&]
 269:        {
 270:           using accscalar_t = acc_type<scalar_t, true>;
 271:           AT_DISPATCH_INDEX_TYPES(indices.scalar_type(), "embedding_dense_backward_cuda", [&] () {
 272:           embedding_backward_feature_kernel<scalar_t, accscalar_t, index_t>
 273:             <<<grid,
 274:                 block,
 275:                 sizeof(accscalar_t)*warp_size*BLOCKDIMY + sizeof(int)*warp_size*BLOCKDIMY,
 276:                 stream>>>
 277:             (indices_contig.const_data_ptr<index_t>(),
 278:               grad.const_data_ptr<scalar_t>(),
 279:               grad_weight.mutable_data_ptr<scalar_t>(),
 280:               static_cast<int>(num_indices),
 281:               static_cast<int64_t>(stride),
 282:               static_cast<int>(padding_idx));
 283:           C10_CUDA_KERNEL_LAUNCH_CHECK();
 284:           });
 285:        });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 286-287
```cpp
 286:     return grad_weight;
 287:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 289-299
```cpp
 289:   auto sorted_indices = at::empty_like(indices, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
 290:   auto orig_indices = at::empty_like(indices, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
 291:   Tensor count;
 292:   AT_DISPATCH_INDEX_TYPES(indices.scalar_type(), "embedding_dense_backward_cuda", [&] () {
 293:     auto range = at::arange(num_indices, indices.options());
 294:     int64_t nbits = cuda::cub::get_num_bits(num_weights);
 295:     cuda::cub::radix_sort_pairs(
 296:       indices.const_data_ptr<index_t>(), sorted_indices.mutable_data_ptr<index_t>(),
 297:       range.const_data_ptr<index_t>(), orig_indices.mutable_data_ptr<index_t>(),
 298:       num_indices, false/*, 0, nbits*/);
 299:   });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 301-322
```cpp
 301:   if (scale_grad_by_freq) {
 302:     count = at::empty_like(indices, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
 303:     AT_DISPATCH_INDEX_TYPES(indices.scalar_type(), "embedding_dense_backward_cuda", [&] () {
 304:       cudaStream_t stream = at::cuda::getCurrentCUDAStream();
 305: 
 306:       // Compute an increasing sequence per unique item in sortedIndices:
 307:       // sorted: 2 5 5 5 7 7 8 9 9
 308:       //  count: 1 1 2 3 1 2 1 1 2
 309:       auto sorted_data = sorted_indices.const_data_ptr<index_t>();
 310:       auto count_data = count.mutable_data_ptr<index_t>();
 311:       cuda::cub::inclusive_sum_by_key(
 312:         sorted_data,
 313:         ATEN_CUB_CONSTANT_ITERATOR(index_t)(1),
 314:         count_data,
 315:         num_indices
 316:       );
 317: 
 318:       // Take the maximum of each count per unique key in reverse:
 319:       // sorted: 2 5 5 5 7 7 8 9 9
 320:       //  count: 1 3 3 3 2 2 1 2 2
 321:       cuda::cub::inclusive_scan_by_key(
 322:         thrust::make_reverse_iterator(sorted_data + num_indices),
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 323-329
```cpp
 323:         thrust::make_reverse_iterator(static_cast<const index_t*>(count_data) + num_indices),
 324:         thrust::make_reverse_iterator(count_data + num_indices),
 325:         ATEN_CUB_MAXIMUM(),
 326:         num_indices
 327:       );
 328:     });
 329:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 331-333
```cpp
 331:   return embedding_backward_cuda_kernel(grad, orig_indices,
 332:       sorted_indices, count, num_weights, padding_idx);
 333: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 335-356
```cpp
 335: Tensor & embedding_renorm_cuda_(Tensor & self, const Tensor & indices,
 336:                                 double max_norm, double norm_type) {
 337:   auto self_arg = TensorArg(self, "self", 1);
 338:   auto indices_arg = TensorArg(indices, "indices", 1);
 339:   checkDim("embedding_renorm_", self_arg, 2);
 340:   checkSameGPU("embedding_renorm", self_arg, indices_arg);
 341: 
 342:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
 343: 
 344:   AT_DISPATCH_INDEX_TYPES(indices.scalar_type(), "embedding_renorm_cuda_", [&] () {
 345: 
 346:     auto num_indices = indices.numel();
 347:     auto indices_contig = std::get<0>(indices.sort()).contiguous();
 348:     auto unique_indices = at::empty(indices.numel(), indices.options());
 349:     auto num_unique_indices = at::empty({}, indices.options().dtype(kLong));
 350: 
 351:     cuda::cub::unique(
 352:       indices_contig.const_data_ptr<index_t>(),
 353:       unique_indices.mutable_data_ptr<index_t>(),
 354:       num_unique_indices.mutable_data_ptr<int64_t>(),
 355:       num_indices
 356:     );
```
- EN: This block defines or continues the implementation of `embedding_renorm_cuda_`.
- CN: 该代码块定义或继续实现 `embedding_renorm_cuda_`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 358-365
```cpp
 358:     int warp_size = at::cuda::warp_size();
 359:     TORCH_INTERNAL_ASSERT(num_threads() % warp_size == 0 &&
 360:                   num_threads() <= static_cast<uint32_t>(cuda_utils::kCUDABlockReduceMaxThreads()),
 361:                   "BlockReduceSum requires all warps be active");
 362:     const int64_t *num_unique_indices_ptr = num_unique_indices.const_data_ptr<int64_t>();
 363:     dim3 grid = unique_indices.numel();
 364:     dim3 block = num_threads();
 365:     int dim = self.stride(0);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 367-380
```cpp
 367:     AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, self.scalar_type(), "embedding_renorm_cuda_", [&] {
 368:       using accscalar_t = acc_type<scalar_t, true>;
 369:       renorm_kernel<<<grid, block, (block.x / warp_size) * sizeof(accscalar_t), stream>>>(
 370:         self.mutable_data_ptr<scalar_t>(),
 371:         unique_indices.const_data_ptr<index_t>(),
 372:         static_cast<accscalar_t>(max_norm),
 373:         static_cast<accscalar_t>(norm_type),
 374:         dim, self.stride(0), self.stride(1),
 375:         num_unique_indices_ptr);
 376:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 377:     });
 378:   });
 379:   return self;
 380: }
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 383-383
```cpp
 383: }  // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- CUB primitives provide parallel CUDA building blocks such as reductions or scans. / CUB 原语提供归约、扫描等并行 CUDA 基础组件。
- Thrust utilities supply STL-like CUDA algorithms and containers. / Thrust 工具提供类似 STL 的 CUDA 算法与容器。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/AccumulateType.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/TensorUtils.h>`
  - `<ATen/ceil_div.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<c10/util/Exception.h>`
  - `<c10/macros/Macros.h>`
  - `<ATen/cuda/cub.cuh>`
  - `<ATen/native/cuda/EmbeddingBackwardKernel.cuh>`
  - `<ATen/native/cuda/SortingCommon.cuh>`
  - `<ATen/native/cuda/block_reduce.cuh>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `AT_DISPATCH_INDEX_TYPES`
  - `cub::get_num_bits`
  - `cub::radix_sort_pairs`
  - `cub::inclusive_sum_by_key`
  - `cub::inclusive_scan_by_key`
  - `cub::unique`
  - `thrust::make_reverse_iterator`
  - `at::cuda::getCurrentCUDAStream`
  - `at::cuda::warp_size`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
