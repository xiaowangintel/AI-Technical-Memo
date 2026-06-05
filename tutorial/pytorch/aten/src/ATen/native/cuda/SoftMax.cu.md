# SoftMax.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/SoftMax.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `log_softmax_cuda_out`, `log_softmax_backward_cuda_out`, `softmax_cuda_out`, `softmax_backward_cuda_out`.
- 用途（中文）: 实现与 `log_softmax_cuda_out`, `log_softmax_backward_cuda_out`, `softmax_cuda_out`, `softmax_backward_cuda_out` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/cuda/CUDAContext.h>
   4: #include <ATen/Dispatch.h>
   5: #include <ATen/TensorUtils.h>
   6: #include <ATen/TensorOperators.h>
   7: #include <ATen/WrapDimUtils.h>
   8: #include <c10/macros/Macros.h>
   9: 
  10: #include <ATen/AccumulateType.h>
  11: #include <ATen/cuda/NumericLimits.cuh>
  12: #include <type_traits>
  13: 
  14: #include <ATen/native/cuda/Loops.cuh>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/cuda/CUDAContext.h>`, `<ATen/Dispatch.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/cuda/CUDAContext.h>`, `<ATen/Dispatch.h>`。

### Lines 15-28
```cpp
  15: #include <ATen/native/cuda/MemoryAccess.cuh>
  16: #include <ATen/native/cuda/PersistentSoftmax.cuh>
  17: #include <ATen/native/IndexingUtils.h>
  18: #include <ATen/native/cuda/block_reduce.cuh>
  19: 
  20: #ifndef AT_PER_OPERATOR_HEADERS
  21: #include <ATen/Functions.h>
  22: #include <ATen/NativeFunctions.h>
  23: #else
  24: #include <ATen/ops/_masked_softmax_native.h>
  25: #include <ATen/ops/_log_softmax_native.h>
  26: #include <ATen/ops/_log_softmax_backward_data_native.h>
  27: #include <ATen/ops/_softmax_native.h>
  28: #include <ATen/ops/_softmax_backward_data_native.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/MemoryAccess.cuh>`, `<ATen/native/cuda/PersistentSoftmax.cuh>`, `<ATen/native/IndexingUtils.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/MemoryAccess.cuh>`, `<ATen/native/cuda/PersistentSoftmax.cuh>`, `<ATen/native/IndexingUtils.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 29-31
```cpp
  29: #include <ATen/ops/softmax.h>
  30: #include <ATen/ops/_softmax_backward_data.h>
  31: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/ops/softmax.h>`, `<ATen/ops/_softmax_backward_data.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/ops/softmax.h>`, `<ATen/ops/_softmax_backward_data.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 33-54
```cpp
  33: namespace at::native {
  34: 
  35: namespace {
  36: 
  37: constexpr int ALIGN_BYTES = 16;
  38: 
  39: template<typename T, typename AccumT, typename OutT>
  40: struct LogSoftMaxForwardEpilogue {
  41:   __device__ __forceinline__ LogSoftMaxForwardEpilogue(AccumT max_input, AccumT sum)
  42:     : max_input(max_input),  logsum(std::log(sum)) {}
  43: 
  44:   __device__ __forceinline__ OutT operator()(T input) const {
  45:     return static_cast<OutT>(input - max_input - logsum);
  46: }
  47: 
  48:   const AccumT max_input;
  49:   const AccumT logsum;
  50: };
  51: 
  52: template<typename T, typename AccumT, typename OutT>
  53: struct LogSoftMaxBackwardEpilogue {
  54:   __device__ __forceinline__ LogSoftMaxBackwardEpilogue(AccumT sum)
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `LogSoftMaxForwardEpilogue`.
- CN: 该代码块定义或继续实现 `LogSoftMaxForwardEpilogue`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 55-55
```cpp
  55:     : sum(sum) {}
```
- EN: This block defines or continues the implementation of `sum`.
- CN: 该代码块定义或继续实现 `sum`。

### Lines 57-59
```cpp
  57:   __device__ __forceinline__ T operator()(OutT gradOutput, OutT output) const {
  58:     return static_cast<T>(gradOutput - std::exp(static_cast<AccumT>(output)) * sum);
  59:   }
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 61-62
```cpp
  61:   const AccumT sum;
  62: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 64-76
```cpp
  64: template<typename T, typename AccumT, typename OutT>
  65: struct SoftMaxForwardEpilogue {
  66:   __device__ __forceinline__ SoftMaxForwardEpilogue(AccumT max_input, AccumT sum)
  67:     : max_input(max_input)
  68:     , sum(sum) {}
  69: 
  70:   __device__ __forceinline__ OutT operator()(T input) const {
  71:     return static_cast<OutT>(std::exp(input - max_input) / sum);
  72:   }
  73: 
  74:   const AccumT max_input;
  75:   const AccumT sum;
  76: };
```
- EN: This block defines or continues the implementation of `SoftMaxForwardEpilogue`.
- CN: 该代码块定义或继续实现 `SoftMaxForwardEpilogue`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 78-90
```cpp
  78: template<typename T, typename AccumT, typename OutT>
  79: struct SoftMaxBackwardEpilogue {
  80:   __device__ __forceinline__ SoftMaxBackwardEpilogue(AccumT sum)
  81:     : sum(sum) {}
  82: 
  83:   // XXX: gradOutput that we get here is really gradOutput * output
  84:   // Look for cmul in SoftMax_updateGradInput
  85:   __device__ __forceinline__ T operator()(OutT gradOutput, OutT output) const {
  86:     return static_cast<T>(gradOutput - output * sum);
  87:   }
  88: 
  89:   const AccumT sum;
  90: };
```
- EN: This block defines or continues the implementation of `SoftMaxBackwardEpilogue`.
- CN: 该代码块定义或继续实现 `SoftMaxBackwardEpilogue`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 92-104
```cpp
  92: template<typename T, typename AccumT, typename OutT>
  93:  struct SoftMaxForwardWithMulEpilogue {
  94:    __device__ __forceinline__ SoftMaxForwardWithMulEpilogue(AccumT max_input, AccumT sum)
  95:      : max_input(max_input)
  96:      , sum(sum) {}
  97: 
  98:    __device__ __forceinline__ OutT operator()(T input) const {
  99:      return static_cast<OutT>(__expf(input - max_input) * sum);
 100:    }
 101: 
 102:    const AccumT max_input;
 103:    const AccumT sum;
 104:  };
```
- EN: This block defines or continues the implementation of `SoftMaxForwardWithMulEpilogue`.
- CN: 该代码块定义或继续实现 `SoftMaxForwardWithMulEpilogue`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 109-117
```cpp
 109: ////////////////////////////////////////////////////////////////////////////////
 110: // Spatial kernel (fast with large inner_size and small dim_size)
 111: ////////////////////////////////////////////////////////////////////////////////
 112: // Let's assume that our input has been flattened to have only three dimension:
 113: //     outer x dim x inner
 114: // The spatial algorithm tries to parallelize along all of them.
 115: // Within a 2d block threadIdx.y parallelizes over dim slices, and threads that
 116: // share it will speed up reductions over dim (along axis x).
 117: // The 2d grid is used to parallelize inner dimension over y axis and outer over x.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 118-130
```cpp
 118: inline dim3 SpatialSoftMax_getGridSize(
 119:     dim3 block, uint32_t max_active_blocks,
 120:     uint64_t outer_size, uint64_t inner_size) {
 121:   // First, tile as many blocks as we can over the y axis
 122:   uint32_t inner_blocks = (inner_size + block.y - 1) / block.y;
 123:   if (inner_blocks > max_active_blocks)
 124:     inner_blocks = max_active_blocks;
 125:   // Fill the x axis with as many blocks as we can fit (a little more is ok too)
 126:   uint32_t outer_blocks = (max_active_blocks + inner_blocks - 1) / inner_blocks;
 127:   if (outer_blocks > outer_size)
 128:     outer_blocks = outer_size;
 129:   return dim3(outer_blocks, inner_blocks);
 130: }
```
- EN: This block defines or continues the implementation of `SpatialSoftMax_getGridSize`.
- CN: 该代码块定义或继续实现 `SpatialSoftMax_getGridSize`。

### Lines 132-132
```cpp
 132: const int max_threads = 1024;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 134-145
```cpp
 134: inline dim3 SpatialSoftMax_getBlockSize(
 135:   uint64_t dim_size, uint64_t inner_size) {
 136:   uint32_t inner_threads = inner_size;
 137:   inner_threads = std::min(inner_threads, static_cast<uint32_t>(max_threads));
 138:   uint32_t dim_threads = 1;
 139:   if (inner_threads <= 64 && dim_size >= 64) {
 140:     while (inner_threads * dim_threads <= max_threads && dim_threads <= dim_size)
 141:       dim_threads *= 2;
 142:     dim_threads /= 2;
 143:   }
 144:   return dim3(dim_threads, inner_threads);
 145: }
```
- EN: This block defines or continues the implementation of `SpatialSoftMax_getBlockSize`.
- CN: 该代码块定义或继续实现 `SpatialSoftMax_getBlockSize`。

### Lines 148-161
```cpp
 148: template<typename accscalar_t, typename Kernel>
 149: void SpatialSoftMax_getLaunchSizes(
 150:     Kernel k,
 151:     uint64_t outer_size, uint64_t dim_size, uint64_t inner_size,
 152:     dim3& grid, dim3& block, uint32_t& smem_size) {
 153:   block = SpatialSoftMax_getBlockSize(dim_size, inner_size);
 154:   uint32_t block_threads = block.x * block.y;
 155:   smem_size = block.x == 1 ? 0 : block_threads * sizeof(accscalar_t);
 156:   int max_active_blocks;
 157:   AT_CUDA_CHECK(cudaOccupancyMaxActiveBlocksPerMultiprocessor(&max_active_blocks,
 158:                                                               k, block_threads, smem_size));
 159:   max_active_blocks *= at::cuda::getCurrentDeviceProperties()->multiProcessorCount;
 160:   grid = SpatialSoftMax_getGridSize(block, max_active_blocks, outer_size, inner_size);
 161: }
```
- EN: This block defines or continues the implementation of `SpatialSoftMax_getLaunchSizes`.
- CN: 该代码块定义或继续实现 `SpatialSoftMax_getLaunchSizes`。

### Lines 163-180
```cpp
 163: inline dim3 SoftMax_getBlockSize(int ILP, uint64_t dim_size) {
 164:   uint64_t block_size = 1;
 165:   uint64_t max_block_size = std::min(dim_size / ILP, static_cast<uint64_t>(max_threads));
 166: 
 167:   // In the vectorized case we want to trade off allowing more of the buffers to be accessed
 168:   // in a vectorized way against wanting a larger block size to get better utilisation.
 169:   // In general with ILP you can have (ILP-1)/ILP of the buffer accessed vectorised, at the risk
 170:   // of having a very small block size. We choose to keep >= 1/2 of the buffer vectorised while
 171:   // allowing a larger block size.
 172:   if (ILP > 1) {
 173:     max_block_size /= 2;
 174:   }
 175: 
 176:   while (block_size < (max_block_size)) block_size *= 2;
 177:   // Launch at least a single warp - the kernel assumes that.
 178:   block_size = std::max(block_size, static_cast<uint64_t>(at::cuda::warp_size()));
 179:   return dim3(block_size);
 180: }
```
- EN: This block defines or continues the implementation of `SoftMax_getBlockSize`, `have`.
- CN: 该代码块定义或继续实现 `SoftMax_getBlockSize`, `have`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 182-199
```cpp
 182: inline dim3 SoftMaxForward_getBlockSize(uint64_t dim_size) {
 183:   uint64_t block_size = 1;
 184:   uint64_t max_block_size = std::min(dim_size, static_cast<uint64_t>(max_threads));
 185: 
 186:   // We need a block size that is a multiple of at::cuda::warp_size() in order
 187:   // to perform block size reductions using warp shuffle instructions.
 188:   // Since max_threads is also a multiple of at::cuda::warp_size() we do not
 189:   // risk creating a block size larger than the limit.
 190: 
 191:   int warp_size = at::cuda::warp_size();
 192:   if (max_block_size % warp_size == 0) {
 193:     block_size = max_block_size;
 194:   } else {
 195:     block_size = (max_block_size / warp_size + 1) * warp_size;
 196:   }
 197: 
 198:   return dim3(block_size);
 199: }
```
- EN: This block defines or continues the implementation of `SoftMaxForward_getBlockSize`.
- CN: 该代码块定义或继续实现 `SoftMaxForward_getBlockSize`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 201-216
```cpp
 201: template<typename T>
 202: struct Add {
 203:   __device__ __forceinline__ T operator()(T a, T b) const {
 204:     return a + b;
 205:   }
 206: 
 207:   __device__ __forceinline__ T combine(T a, T b) const {
 208:     return a + b;
 209:   }
 210: 
 211:   // Needed to allow warp level reduction as a first step in the
 212:   // thread block reduction
 213:   __device__ __forceinline__ T warp_shfl_down(T data, int offset) const {
 214:     return WARP_SHFL_DOWN(data, offset);
 215:   }
 216: };
```
- EN: This block defines or continues the implementation of `combine`, `warp_shfl_down`.
- CN: 该代码块定义或继续实现 `combine`, `warp_shfl_down`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 218-233
```cpp
 218: template<typename T>
 219: struct Max {
 220:   __device__ __forceinline__ T operator()(T a, T b) const {
 221:     return a < b ? b : a;
 222:   }
 223: 
 224:   __device__ __forceinline__ T combine(T a, T b) const {
 225:     return a < b ? b : a;
 226:   }
 227: 
 228:   // Needed to allow warp level reduction as a first step in the
 229:   // thread block reduction
 230:   __device__ __forceinline__ T warp_shfl_down(T data, int offset) const {
 231:     return WARP_SHFL_DOWN(data, offset);
 232:   }
 233: };
```
- EN: This block defines or continues the implementation of `combine`, `warp_shfl_down`.
- CN: 该代码块定义或继续实现 `combine`, `warp_shfl_down`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 235-236
```cpp
 235: // Note that it's not a complete block-wide reduction.
 236: // Only threads that share threadIdx.y reduce values.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 237-258
```cpp
 237: template<typename T, template<typename> class ReduceOp>
 238: __forceinline__ __device__
 239: T spatialBlockReduceX(T *shared, T val) {
 240:   ReduceOp<T> r;
 241:   shared += threadIdx.y * blockDim.x;
 242: 
 243:   __syncthreads();
 244: 
 245:   shared[threadIdx.x] = val;
 246: 
 247:   // NOTE: loop starts with __syncthreads()
 248:   int offset = blockDim.x / 2;
 249:   while (offset > 0) {
 250:     __syncthreads();
 251:     if (threadIdx.x < offset)
 252:       shared[threadIdx.x] = r(shared[threadIdx.x], shared[threadIdx.x + offset]);
 253:     offset /= 2;
 254:   }
 255: 
 256:   __syncthreads();
 257: 
 258:   return shared[0];
```
- EN: This block defines or continues the implementation of `spatialBlockReduceX`.
- CN: 该代码块定义或继续实现 `spatialBlockReduceX`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 259-259
```cpp
 259: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 261-282
```cpp
 261: template <typename scalar_t, typename accscalar_t, typename outscalar_t, typename index_t, template<typename, typename, typename> class Epilogue>
 262: __global__ void cunn_SpatialSoftMaxForward(
 263:     outscalar_t *output, const scalar_t *input,
 264:     index_t outer_size, index_t dim_size, index_t inner_size)
 265: {
 266:   extern __shared__ unsigned char smem[];
 267:   auto sdata = reinterpret_cast<accscalar_t*>(smem);
 268:   const index_t outer_stride = inner_size * dim_size;
 269:   const index_t dim_stride = inner_size;
 270: 
 271:   for (index_t outer_index = blockIdx.x; outer_index < outer_size; outer_index += gridDim.x) {
 272:     const index_t outer_offset = outer_index * outer_stride;
 273:     for (index_t inner_index = blockIdx.y * blockDim.y + threadIdx.y; inner_index < inner_size; inner_index += blockDim.y * gridDim.y) {
 274:       const index_t data_offset = outer_offset + inner_index;
 275:       ////////////////////////////////////////////////////////////
 276:       // These two blocks are really equivalent, but specializing on
 277:       // blockDim.x == 1 makes the kernel faster when it's unused.
 278:       // I didn't want to thread an extra template parameter, and nvcc
 279:       // seems to be smart enough to hoist the if outside of the loops.
 280:       ////////////////////////////////////////////////////////////
 281: 
 282:       if (blockDim.x > 1) {
```
- EN: This block defines GPU kernel entry point(s) `cunn_SpatialSoftMaxForward`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `cunn_SpatialSoftMaxForward`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 283-288
```cpp
 283:         accscalar_t max_input = std::numeric_limits<accscalar_t>::lowest();
 284:         for (index_t d = threadIdx.x; d < dim_size; d += blockDim.x) {
 285:           const accscalar_t value = static_cast<accscalar_t>(input[data_offset + d * dim_stride]);
 286:           max_input = Max<accscalar_t>()(max_input, value);
 287:         }
 288:         max_input = spatialBlockReduceX<accscalar_t, Max>(sdata,max_input);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 290-294
```cpp
 290:         accscalar_t sum = 0;
 291:         for (index_t d = threadIdx.x; d < dim_size; d += blockDim.x)
 292:           sum += std::exp(static_cast<accscalar_t>(input[data_offset + d * dim_stride])
 293:                  - max_input);
 294:         sum = spatialBlockReduceX<accscalar_t, Add>(sdata, sum);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 296-309
```cpp
 296:         Epilogue<scalar_t, accscalar_t, outscalar_t> epilogue(max_input, sum);
 297:         for (index_t d = threadIdx.x; d < dim_size; d += blockDim.x)
 298:           output[data_offset + d * dim_stride] = epilogue(input[data_offset + d * dim_stride]);
 299:       } else {
 300:         accscalar_t max_input = std::numeric_limits<accscalar_t>::lowest();
 301:         for (index_t d = threadIdx.x; d < dim_size; d += blockDim.x) {
 302:           const accscalar_t value = static_cast<accscalar_t>(input[data_offset + d * dim_stride]);
 303:           max_input = Max<accscalar_t>()(max_input, value);
 304:         }
 305:         accscalar_t sum = 0;
 306:         for (index_t d = threadIdx.x; d < dim_size; d += blockDim.x)
 307:           sum += std::exp(static_cast<accscalar_t>(input[data_offset + d * dim_stride])
 308:                  - max_input);
 309:         Epilogue<scalar_t, accscalar_t, outscalar_t> epilogue(max_input, sum);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 310-315
```cpp
 310:         for (index_t d = threadIdx.x; d < dim_size; d += blockDim.x)
 311:           output[data_offset + d * dim_stride] = epilogue(input[data_offset + d * dim_stride]);
 312:       }
 313:     }
 314:   }
 315: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 319-340
```cpp
 319: template <typename scalar_t, typename accscalar_t, typename outscalar_t, template<typename, typename, typename> class Epilogue>
 320: __global__ void cunn_SpatialSoftMaxBackward(
 321:     scalar_t *gradInput, const outscalar_t *output, const outscalar_t *gradOutput,
 322:     uint32_t outer_size, uint32_t dim_size, uint32_t inner_size)
 323: {
 324:   extern __shared__ unsigned char smem[];
 325:   auto sdata = reinterpret_cast<accscalar_t*>(smem);
 326:   const uint32_t outer_stride = inner_size * dim_size;
 327:   const uint32_t dim_stride = inner_size;
 328: 
 329:   for (uint32_t outer_index = blockIdx.x; outer_index < outer_size; outer_index += gridDim.x) {
 330:     const uint32_t outer_offset = outer_index * outer_stride;
 331:     for (uint32_t inner_index = blockIdx.y * blockDim.y + threadIdx.y; inner_index < inner_size; inner_index += blockDim.y * gridDim.y) {
 332:       const uint32_t data_offset = outer_offset + inner_index;
 333:       // See the comment in forward kernel
 334:       if (blockDim.x > 1) {
 335:         accscalar_t sum = 0;
 336:         for (uint32_t d = threadIdx.x; d < dim_size; d += blockDim.x)
 337:           sum += gradOutput[data_offset + d * dim_stride];
 338:         sum = spatialBlockReduceX<accscalar_t, Add>(sdata, sum);
 339: 
 340:         Epilogue<scalar_t, accscalar_t, outscalar_t> epilogue(sum);
```
- EN: This block defines GPU kernel entry point(s) `cunn_SpatialSoftMaxBackward`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `cunn_SpatialSoftMaxBackward`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 341-349
```cpp
 341:         for (uint32_t d = threadIdx.x; d < dim_size; d += blockDim.x) {
 342:           gradInput[data_offset + d * dim_stride] =
 343:             epilogue(gradOutput[data_offset + d * dim_stride],
 344:                     output[data_offset + d * dim_stride]);
 345:         }
 346:       } else {
 347:         accscalar_t sum = 0;
 348:         for (uint32_t d = 0; d < dim_size; d++)
 349:           sum += gradOutput[data_offset + d * dim_stride];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 351-360
```cpp
 351:         Epilogue<scalar_t, accscalar_t, outscalar_t> epilogue(sum);
 352:         for (uint32_t d = 0; d < dim_size; d++) {
 353:           gradInput[data_offset + d * dim_stride] =
 354:             epilogue(gradOutput[data_offset + d * dim_stride],
 355:                     output[data_offset + d * dim_stride]);
 356:         }
 357:       }
 358:     }
 359:   }
 360: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 363-365
```cpp
 363: ////////////////////////////////////////////////////////////////////////////////
 364: // Regular kernel (fast when dim_size is large; requires inner_size == 1)
 365: ////////////////////////////////////////////////////////////////////////////////
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 368-374
```cpp
 368: template <typename T, typename AccumT>
 369: struct MaxFloat
 370: {
 371:   __device__ __forceinline__ AccumT operator()(AccumT max, T v) const {
 372:     return ::max(max, (AccumT)v);
 373:   }
 374: };
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 376-382
```cpp
 376: template<typename T, typename AccumT>
 377: struct AddFloat
 378: {
 379:   __device__ __forceinline__ AccumT operator()(AccumT sum, T v) const {
 380:     return sum + v;
 381:   }
 382: };
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 384-395
```cpp
 384: template<typename T, typename AccumT>
 385: struct SumExpFloat
 386: {
 387:   __device__ __forceinline__ SumExpFloat(AccumT v)
 388:     : max_k(v) {}
 389: 
 390:   __device__ __forceinline__ AccumT operator()(AccumT sum, T v) const {
 391:     return sum + std::exp(v - max_k);
 392:   }
 393: 
 394:   const AccumT max_k;
 395: };
```
- EN: This block defines or continues the implementation of `SumExpFloat`.
- CN: 该代码块定义或继续实现 `SumExpFloat`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 397-408
```cpp
 397: template<typename T, typename AccumT>
 398: struct SumExpfFloat
 399: {
 400:   __device__ __forceinline__ SumExpfFloat(AccumT v)
 401:     : max_k(v) {}
 402: 
 403:   __device__ __forceinline__ AccumT operator()(AccumT sum, T v) const {
 404:     return sum + __expf(v - max_k);
 405:   }
 406: 
 407:   const AccumT max_k;
 408: };
```
- EN: This block defines or continues the implementation of `SumExpfFloat`.
- CN: 该代码块定义或继续实现 `SumExpfFloat`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 410-431
```cpp
 410: template <template<typename> class Reduction, typename AccumT>
 411: __device__ __forceinline__ AccumT
 412: blockReduce(AccumT* smem, AccumT val,
 413:             const Reduction<AccumT>& r,
 414:             AccumT defaultVal)
 415: {
 416:   // To avoid RaW races from chaining blockReduce calls together, we need a sync here
 417:   __syncthreads();
 418: 
 419:   smem[threadIdx.x] = val;
 420: 
 421:   __syncthreads();
 422: 
 423:   AccumT warpVal = defaultVal;
 424: 
 425:   // First warp will perform per-warp reductions for the remaining warps
 426:   uint32_t mask = (((uint64_t)1) << (blockDim.x / C10_WARP_SIZE)) - 1;
 427:   if (threadIdx.x < C10_WARP_SIZE) {
 428:     int lane = threadIdx.x % C10_WARP_SIZE;
 429:     if (lane < blockDim.x / C10_WARP_SIZE) {
 430: #pragma unroll
 431:       for (int i = 0; i < C10_WARP_SIZE; ++i) {
```
- EN: This block defines or continues the implementation of `blockReduce`.
- CN: 该代码块定义或继续实现 `blockReduce`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 432-439
```cpp
 432:         warpVal = r(warpVal, smem[lane * C10_WARP_SIZE + i]);
 433:       }
 434: #if !defined(USE_ROCM)
 435:       __syncwarp(mask);
 436: #endif
 437:       smem[lane] = warpVal;
 438:     }
 439:   }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 441-441
```cpp
 441:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 443-443
```cpp
 443:   // First thread will perform a reduction of the above per-warp reductions
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 444-444
```cpp
 444:   AccumT blockVal = defaultVal;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 446-451
```cpp
 446:   if (threadIdx.x == 0) {
 447:     for (int i = 0; i < blockDim.x / C10_WARP_SIZE; ++i) {
 448:       blockVal = r(blockVal, smem[i]);
 449:     }
 450:     smem[0] = blockVal;
 451:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 453-453
```cpp
 453:   // Sync and broadcast
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 454-456
```cpp
 454:   __syncthreads();
 455:   return smem[0];
 456: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 458-459
```cpp
 458: // Performs a thread block reduction with a given functor but uses
 459: // warp shuffles as the first step in the reduction
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 460-470
```cpp
 460: template <template<typename> class Reduction, typename T>
 461: __device__ __forceinline__
 462: T blockReduceWarp(T* smem_cache, T value, const Reduction<T>& op, T defaultVal)
 463: {
 464:   T result = cuda_utils::BlockReduce<T, Reduction<T>>(value, op, defaultVal, smem_cache);
 465:   if (threadIdx.x == 0) {
 466:     smem_cache[0] = result;
 467:   }
 468:   __syncthreads();
 469:   return smem_cache[0];
 470: }
```
- EN: This block defines or continues the implementation of `blockReduceWarp`.
- CN: 该代码块定义或继续实现 `blockReduceWarp`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 473-483
```cpp
 473: template <template<typename> class Reduction, typename T>
 474: __device__ __forceinline__
 475: T blockReduceWarpInverse(T* smem_cache, T value, const Reduction<T>& op, T defaultVal)
 476: {
 477:   T result = cuda_utils::BlockReduce<T, Reduction<T>>(value, op, defaultVal, smem_cache);
 478:   if (threadIdx.x == 0) {
 479:     smem_cache[0] = 1 / result;
 480:   }
 481:   __syncthreads();
 482:   return smem_cache[0];
 483: }
```
- EN: This block defines or continues the implementation of `blockReduceWarpInverse`.
- CN: 该代码块定义或继续实现 `blockReduceWarpInverse`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 485-506
```cpp
 485: template <template<typename, typename> class Reduction, int ILP, typename T, typename AccumT, typename index_t=int>
 486: __device__ __forceinline__ AccumT
 487: ilpReduce(index_t shift,
 488:           const T* data,
 489:           index_t size,
 490:           const Reduction<T, AccumT>& r,
 491:           AccumT defaultVal)
 492: {
 493:   using LoadT = at::native::memory::aligned_vector<T, ILP>;
 494:   AccumT threadVal = defaultVal;
 495:   index_t offset = threadIdx.x;
 496: 
 497:   // shift and do 1
 498:   if(shift > 0){
 499:     data -= shift;
 500:     size += shift;
 501:     if (offset >= shift && offset < size) {
 502:       threadVal = r(threadVal, data[offset]);
 503:     }
 504:     size -= blockDim.x > size ? size : blockDim.x;
 505:     data += blockDim.x;
 506:   }
```
- EN: This block defines or continues the implementation of `ilpReduce`.
- CN: 该代码块定义或继续实现 `ilpReduce`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 507-507
```cpp
 507:   index_t last = size % (ILP * blockDim.x);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 509-510
```cpp
 509:   T v[ILP];
 510:   LoadT* value = reinterpret_cast<LoadT*>(&v);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 512-519
```cpp
 512:   for (; offset * ILP < (size - last); offset += blockDim.x) {
 513:     *value = reinterpret_cast<const LoadT*>(data)[offset];
 514: 
 515:     #pragma unroll
 516:     for (int j = 0; j < ILP; ++j) {
 517:       threadVal = r(threadVal, v[j]);
 518:     }
 519:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 521-524
```cpp
 521:   offset = size - last + threadIdx.x;
 522:   // Epilogue
 523:   for (; offset < size; offset += blockDim.x)
 524:     threadVal = r(threadVal, data[offset]);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 526-527
```cpp
 526:   return threadVal;
 527: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 529-533
```cpp
 529: int32_t potential_register_count(int32_t dim_size, int32_t thread_count){
 530:   // This method calculate the potential register count for ilpReduce method (it's just a rough number).
 531:   int reg_cnt = (dim_size + thread_count - 1) / thread_count;
 532:   return reg_cnt;
 533: }
```
- EN: This block defines or continues the implementation of `potential_register_count`.
- CN: 该代码块定义或继续实现 `potential_register_count`。

### Lines 535-537
```cpp
 535: /**
 536:  * This will apply the Epilogue with vectorized reads & writes when input & output have the same shift
 537:  */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 538-559
```cpp
 538: template <int ILP, typename scalar_t, typename accum_t, typename outscalar_t, template<typename, typename, typename> class Epilogue>
 539: __device__ __forceinline__ void
 540: WriteFpropResultsVectorized(
 541:              int size,
 542:              const int shift,
 543:              const scalar_t *input,
 544:              outscalar_t *output,
 545:              Epilogue<scalar_t, accum_t, outscalar_t> epilogue) {
 546:   using LoadT = at::native::memory::aligned_vector<scalar_t, ILP>;
 547:   using StoreT = at::native::memory::aligned_vector<outscalar_t, ILP>;
 548: 
 549:   int offset = threadIdx.x;
 550: 
 551:   // if unaligned, do one value / thread and move on, guaranteeing aligned reads/writes later
 552:   if (shift > 0) {
 553:     input -= shift;
 554:     output -= shift;
 555:     size += shift;
 556: 
 557:     if (offset >= shift && offset < size) {
 558:       output[offset] = epilogue(input[offset]);
 559:     }
```
- EN: This block defines or continues the implementation of `WriteFpropResultsVectorized`.
- CN: 该代码块定义或继续实现 `WriteFpropResultsVectorized`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 560-563
```cpp
 560:     size -= blockDim.x > size ? size : blockDim.x;
 561:     input += blockDim.x;
 562:     output += blockDim.x;
 563:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 565-565
```cpp
 565:   const int last = size % (ILP * blockDim.x);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 567-568
```cpp
 567:   scalar_t in_v[ILP];
 568:   LoadT* in_value = reinterpret_cast<LoadT*>(&in_v);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 570-571
```cpp
 570:   outscalar_t out_v[ILP];
 571:   const StoreT* out_value = reinterpret_cast<const StoreT*>(&out_v);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 573-582
```cpp
 573:   for (; offset * ILP < (size - last); offset += blockDim.x) {
 574:     *in_value = reinterpret_cast<const LoadT*>(input)[offset];
 575: 
 576:     #pragma unroll
 577:     for (int j = 0; j < ILP; ++j) {
 578:       out_v[j] = epilogue(in_v[j]);
 579:     }
 580: 
 581:     reinterpret_cast<StoreT*>(output)[offset] = *out_value;
 582:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 584-589
```cpp
 584:   offset = size - last + threadIdx.x;
 585:   // handle the tail
 586:   for (; offset < size; offset += blockDim.x) {
 587:     output[offset] = epilogue(input[offset]);
 588:   }
 589: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 591-612
```cpp
 591: template <int ILP, typename scalar_t, typename accum_t, typename outscalar_t, template<typename, typename, typename> class Epilogue, typename index_t = int32_t>
 592: __device__ __forceinline__ void
 593: WriteBpropResultsVectorized(
 594:              index_t size,
 595:              const index_t shift,
 596:              scalar_t *gradInput,
 597:              const outscalar_t *output,
 598:              const outscalar_t *gradOutput,
 599:              Epilogue<scalar_t, accum_t, outscalar_t> epilogue) {
 600:   using gradInputT = at::native::memory::aligned_vector<scalar_t, ILP>;
 601:   using outputT = at::native::memory::aligned_vector<outscalar_t, ILP>;
 602: 
 603:   index_t offset = threadIdx.x;
 604: 
 605:   // if unaligned, do one value / thread and move on, guaranteeing aligned reads/writes later
 606:   if (shift > 0) {
 607:     gradInput -= shift;
 608:     output -= shift;
 609:     gradOutput -= shift;
 610:     size += shift;
 611: 
 612:     if (threadIdx.x >= shift) {
```
- EN: This block defines or continues the implementation of `WriteBpropResultsVectorized`.
- CN: 该代码块定义或继续实现 `WriteBpropResultsVectorized`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 613-619
```cpp
 613:       gradInput[offset] = epilogue(gradOutput[offset], output[offset]);
 614:     }
 615:     size -= blockDim.x > size ? size : blockDim.x;
 616:     gradInput += blockDim.x;
 617:     output += blockDim.x;
 618:     gradOutput += blockDim.x;
 619:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 621-621
```cpp
 621:   const index_t last = size % (ILP * blockDim.x);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 623-624
```cpp
 623:   scalar_t dX[ILP];
 624:   gradInputT *dX_v = reinterpret_cast<gradInputT*>(&dX);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 626-627
```cpp
 626:   outscalar_t Y[ILP];
 627:   outputT *Y_v = reinterpret_cast<outputT*>(&Y);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 629-630
```cpp
 629:   outscalar_t dY[ILP];
 630:   outputT *dY_v = reinterpret_cast<outputT*>(&dY);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 632-642
```cpp
 632:   for (; offset * ILP < (size - last); offset += blockDim.x) {
 633:     *Y_v = reinterpret_cast<const outputT*>(output)[offset];
 634:     *dY_v = reinterpret_cast<const outputT*>(gradOutput)[offset];
 635: 
 636:     #pragma unroll
 637:     for (int j = 0; j < ILP; ++j) {
 638:       dX[j] = epilogue(dY[j], Y[j]);
 639:     }
 640: 
 641:     reinterpret_cast<gradInputT*>(gradInput)[offset] = *dX_v;
 642:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 644-648
```cpp
 644:   offset = size - last + threadIdx.x;
 645:   for (; offset < size; offset += blockDim.x) {
 646:     gradInput[offset] = epilogue(gradOutput[offset], output[offset]);
 647:   }
 648: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 650-652
```cpp
 650: /**
 651:  * This will apply the Epilogue with non-vectorized reads & writes for the general case
 652:  */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 653-663
```cpp
 653: template <int ILP, typename scalar_t, typename accum_t, typename outscalar_t, template<typename, typename, typename> class Epilogue>
 654: __device__ __forceinline__ void
 655: WriteFpropResults(
 656:              int classes,
 657:              const scalar_t *input,
 658:              outscalar_t *output,
 659:              Epilogue<scalar_t, accum_t, outscalar_t> epilogue) {
 660:   for (int offset = threadIdx.x; offset < classes; offset += blockDim.x) {
 661:     output[offset] = epilogue(input[offset]);
 662:   }
 663: }
```
- EN: This block defines or continues the implementation of `WriteFpropResults`.
- CN: 该代码块定义或继续实现 `WriteFpropResults`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 665-686
```cpp
 665: template <int ILP, typename scalar_t, typename accum_t, typename outscalar_t, template<typename, typename, typename> class Epilogue, typename index_t>
 666: __device__ __forceinline__ void
 667: WriteBpropResults(
 668:              int classes,
 669:              scalar_t *gradInput,
 670:              const outscalar_t *output,
 671:              const outscalar_t *gradOutput,
 672:              Epilogue<scalar_t, accum_t, outscalar_t> epilogue) {
 673: 
 674:   index_t offset = threadIdx.x;
 675: 
 676:   index_t last = classes % (ILP * blockDim.x);
 677: 
 678:   for (; offset < classes - last; offset += blockDim.x * ILP) {
 679:     outscalar_t tmpOutput[ILP];
 680:     outscalar_t tmpGradOutput[ILP];
 681: 
 682:     #pragma unroll
 683:     for (int j = 0; j < ILP; ++j) {
 684:       tmpOutput[j] = output[offset + j * blockDim.x];
 685:       tmpGradOutput[j] = gradOutput[offset + j * blockDim.x];
 686:     }
```
- EN: This block defines or continues the implementation of `WriteBpropResults`.
- CN: 该代码块定义或继续实现 `WriteBpropResults`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 688-688
```cpp
 688:     #pragma unroll
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 689-692
```cpp
 689:     for (int j = 0; j < ILP; ++j) {
 690:       gradInput[offset + j * blockDim.x] = epilogue(tmpGradOutput[j], tmpOutput[j]);
 691:     }
 692:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 694-694
```cpp
 694:   // Remainder - no ILP
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 695-698
```cpp
 695:   for (; offset < classes; offset += blockDim.x) {
 696:     gradInput[offset] = epilogue(gradOutput[offset], output[offset]);
 697:   }
 698: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 700-721
```cpp
 700: template <int ILP, typename scalar_t, typename accscalar_t, typename outscalar_t, template <typename, typename, typename> class EpilogueWithMul>
 701: __global__ void
 702: cunn_SoftMaxForwardFast(outscalar_t *output, const scalar_t *input, int classes)
 703: {
 704:   extern __shared__ unsigned char smem[];
 705:   auto sdata = reinterpret_cast<accscalar_t*>(smem);
 706: 
 707:   // each block handles a sample in the mini-batch
 708:   input += static_cast<int64_t>(blockIdx.x) * classes;
 709:   output += static_cast<int64_t>(blockIdx.x) * classes;
 710: 
 711:   const int shift = ((uint64_t)input) % ALIGN_BYTES / sizeof(scalar_t);
 712: 
 713:   // find the max
 714:   accscalar_t threadMax = ilpReduce<MaxFloat, ILP, scalar_t, accscalar_t>(
 715:     shift, input, classes, MaxFloat<scalar_t, accscalar_t>(), std::numeric_limits<accscalar_t>::lowest());
 716:   accscalar_t max_k = blockReduceWarp<Max, accscalar_t>(sdata, threadMax,
 717:     Max<accscalar_t>(), std::numeric_limits<accscalar_t>::lowest());
 718: 
 719:   // reduce all values
 720:   accscalar_t threadExp = ilpReduce<SumExpfFloat, ILP, scalar_t, accscalar_t>(
 721:     shift, input, classes, SumExpfFloat<scalar_t, accscalar_t>(max_k), static_cast<accscalar_t>(0));
```
- EN: This block defines GPU kernel entry point(s) `cunn_SoftMaxForwardFast`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `cunn_SoftMaxForwardFast`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 722-723
```cpp
 722:   accscalar_t sumAll = blockReduceWarpInverse<Add, accscalar_t>(sdata, threadExp,
 723:     Add<accscalar_t>(), static_cast<accscalar_t>(0));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 725-725
```cpp
 725:   EpilogueWithMul<scalar_t, accscalar_t, outscalar_t> epilogue(max_k, sumAll);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 727-730
```cpp
 727:   for (int offset = threadIdx.x; offset < classes; offset += blockDim.x) {
 728:     output[offset] = epilogue(input[offset]);
 729:   }
 730: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 732-753
```cpp
 732: template <int ILP, typename scalar_t, typename accscalar_t, typename outscalar_t, template <typename, typename, typename> class Epilogue>
 733: __global__ void
 734: cunn_SoftMaxForward(outscalar_t *output, const scalar_t *input, int classes)
 735: {
 736:   extern __shared__ unsigned char smem[];
 737:   auto sdata = reinterpret_cast<accscalar_t*>(smem);
 738: 
 739:   // forward pointers to batch[blockIdx.x]
 740:   // each block handles a sample in the mini-batch
 741:   input += static_cast<int64_t>(blockIdx.x) * classes;
 742:   output += static_cast<int64_t>(blockIdx.x) * classes;
 743: 
 744:   const int shift = ((uint64_t)input) % ALIGN_BYTES / sizeof(scalar_t);
 745:   const int output_shift = ((uint64_t)output) % ALIGN_BYTES / sizeof(outscalar_t);
 746: 
 747:   // find the max
 748:   accscalar_t threadMax = ilpReduce<MaxFloat, ILP, scalar_t, accscalar_t>(
 749:     shift, input, classes, MaxFloat<scalar_t, accscalar_t>(), std::numeric_limits<accscalar_t>::lowest());
 750:   accscalar_t max_k = blockReduceWarp<Max, accscalar_t>(sdata, threadMax,
 751:     Max<accscalar_t>(), std::numeric_limits<accscalar_t>::lowest());
 752: 
 753:   // reduce all values
```
- EN: This block defines GPU kernel entry point(s) `cunn_SoftMaxForward`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `cunn_SoftMaxForward`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 754-757
```cpp
 754:   accscalar_t threadExp = ilpReduce<SumExpFloat, ILP, scalar_t, accscalar_t>(
 755:     shift, input, classes, SumExpFloat<scalar_t, accscalar_t>(max_k), static_cast<accscalar_t>(0));
 756:   accscalar_t sumAll = blockReduceWarp<Add, accscalar_t>(sdata, threadExp,
 757:     Add<accscalar_t>(), static_cast<accscalar_t>(0));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 759-759
```cpp
 759:   Epilogue<scalar_t, accscalar_t, outscalar_t> epilogue(max_k, sumAll);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 761-766
```cpp
 761:   if (shift == output_shift) {
 762:     WriteFpropResultsVectorized<ILP, scalar_t, accscalar_t, outscalar_t, Epilogue>(classes, shift, input, output, epilogue);
 763:   } else {
 764:     WriteFpropResults<ILP, scalar_t, accscalar_t, outscalar_t, Epilogue>(classes, input, output, epilogue);
 765:   }
 766: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 768-789
```cpp
 768: template <typename scalar_t, typename accscalar_t, typename outscalar_t, template <typename, typename, typename> class Epilogue, typename index_t, int32_t reg_cnt>
 769: __global__ void
 770: cunn_SoftMaxForwardReg(outscalar_t *output, const scalar_t *input, index_t classes)
 771: {
 772:   extern __shared__ unsigned char smem[];
 773:   auto sdata = reinterpret_cast<accscalar_t*>(smem);
 774: 
 775:   scalar_t reg[reg_cnt];
 776: 
 777:   input += static_cast<int64_t>(blockIdx.x) * classes;
 778:   output += static_cast<int64_t>(blockIdx.x) * classes;
 779: 
 780:   accscalar_t threadMax = std::numeric_limits<accscalar_t>::lowest();
 781:   accscalar_t threadExp = static_cast<accscalar_t>(0);
 782: 
 783:   // Load the elements from gmem into reg, and get the max for current thread.
 784:   MaxFloat<scalar_t, accscalar_t> maxFunc;
 785: 
 786:   #pragma unroll
 787:   for(int reg_idx = 0; reg_idx < reg_cnt; reg_idx ++){
 788:     int offset = threadIdx.x + reg_idx * blockDim.x;
 789:     if(offset < classes) {
```
- EN: This block defines GPU kernel entry point(s) `cunn_SoftMaxForwardReg`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `cunn_SoftMaxForwardReg`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 790-793
```cpp
 790:       reg[reg_idx] = input[offset];
 791:       threadMax = maxFunc(threadMax, reg[reg_idx]);
 792:     }
 793:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 795-795
```cpp
 795:   // Reduce to the max for block
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 796-797
```cpp
 796:   accscalar_t max_k = blockReduceWarp<Max, accscalar_t>(sdata, threadMax,
 797:     Max<accscalar_t>(), std::numeric_limits<accscalar_t>::lowest());
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 799-809
```cpp
 799:   SumExpFloat<scalar_t, accscalar_t> sumExpFunc(max_k);
 800:   // reduce all values
 801:   #pragma unroll
 802:   for(int reg_idx = 0; reg_idx < reg_cnt; reg_idx ++){
 803:     int offset = threadIdx.x + reg_idx * blockDim.x;
 804:     if(offset < classes) {
 805:       threadExp = sumExpFunc(threadExp, reg[reg_idx]);
 806:     }
 807:   }
 808:   accscalar_t sumAll = blockReduceWarp<Add, accscalar_t>(sdata, threadExp,
 809:     Add<accscalar_t>(), static_cast<accscalar_t>(0));
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 811-811
```cpp
 811:   Epilogue<scalar_t, accscalar_t, outscalar_t> epilogue(max_k, sumAll);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 813-813
```cpp
 813:   // Write back the value
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 814-814
```cpp
 814:   #pragma unroll
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 815-821
```cpp
 815:   for(int reg_idx = 0; reg_idx < reg_cnt; reg_idx ++){
 816:     int offset = threadIdx.x + reg_idx * blockDim.x;
 817:     if(offset < classes) {
 818:       output[offset] = epilogue(reg[reg_idx]);
 819:     }
 820:   }
 821: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 824-845
```cpp
 824: template <int ILP, typename scalar_t, typename accscalar_t, typename outscalar_t,
 825:   template <typename, typename, typename> class EpilogueWithMul, typename index_t = int32_t>
 826: __global__ void
 827: cunn_SoftMaxForwardGmem(outscalar_t *output, const scalar_t *input, index_t classes)
 828: {
 829:   // Each thread block processes a sample in the batch
 830:   input += static_cast<int64_t>(blockIdx.x) * classes;
 831:   output += static_cast<int64_t>(blockIdx.x) * classes;
 832: 
 833:   accscalar_t threadMax = std::numeric_limits<accscalar_t>::lowest();
 834:   accscalar_t threadExp = static_cast<accscalar_t>(0);
 835: 
 836:   // The first smem segment is used to cache input values and the last
 837:   // segment is used for thread block reductions
 838:   extern __shared__ unsigned char smem[];
 839:   auto smem_reduction_cache = reinterpret_cast<accscalar_t*>(smem);
 840: 
 841:   using LoadT = at::native::memory::aligned_vector<scalar_t, ILP>;
 842:   const LoadT* const input_vec_ptr = reinterpret_cast<const LoadT*>(input);
 843: 
 844:   // Do the first step in max calculation:
 845:   MaxFloat<scalar_t, accscalar_t> maxFunc;
```
- EN: This block defines GPU kernel entry point(s) `cunn_SoftMaxForwardGmem`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `cunn_SoftMaxForwardGmem`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 846-852
```cpp
 846:   for (index_t offset = threadIdx.x; offset * ILP < classes; offset += blockDim.x) {
 847:     LoadT crnt_vec = input_vec_ptr[offset];
 848:     #pragma unroll
 849:     for (int i = 0; i < ILP; ++i) {
 850:       threadMax = maxFunc(threadMax, crnt_vec.val[i]);
 851:     }
 852:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 854-855
```cpp
 854:   accscalar_t max_k = blockReduceWarp<Max, accscalar_t>(smem_reduction_cache, threadMax,
 855:     Max<accscalar_t>(), std::numeric_limits<accscalar_t>::lowest());
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 857-857
```cpp
 857:   // Do the second step in sum exp calculation:
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 858-865
```cpp
 858:   SumExpfFloat<scalar_t, accscalar_t> sumExpFunc(max_k);
 859:   for (index_t offset = threadIdx.x; offset * ILP < classes; offset += blockDim.x) {
 860:     LoadT crnt_vec = input_vec_ptr[offset];
 861:     #pragma unroll
 862:     for (int i = 0; i < ILP; ++i) {
 863:       threadExp = sumExpFunc(threadExp, crnt_vec.val[i]);
 864:     }
 865:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 867-868
```cpp
 867:   accscalar_t sumAll = blockReduceWarpInverse<Add, accscalar_t>(smem_reduction_cache, threadExp,
 868:     Add<accscalar_t>(), static_cast<accscalar_t>(0));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 870-870
```cpp
 870:   EpilogueWithMul<scalar_t, accscalar_t, outscalar_t> epilogue(max_k, sumAll);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 872-883
```cpp
 872:   using StoreT = at::native::memory::aligned_vector<outscalar_t, ILP>;
 873:   StoreT* output_vec_ptr = reinterpret_cast<StoreT*>(output);
 874:   for (index_t offset = threadIdx.x; offset * ILP < classes; offset += blockDim.x) {
 875:     LoadT crnt_vec = input_vec_ptr[offset];
 876:     StoreT out_vec;
 877:     #pragma unroll
 878:     for (int i = 0; i < ILP; ++i) {
 879:       out_vec.val[i] = epilogue(crnt_vec.val[i]);
 880:     }
 881:     output_vec_ptr[offset] = out_vec;
 882:   }
 883: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 885-906
```cpp
 885: template <int ILP, typename scalar_t, typename accscalar_t, typename outscalar_t,
 886:   template <typename, typename, typename> class Epilogue, typename index_t = int32_t>
 887: __global__ void
 888: cunn_SoftMaxForwardSmem(outscalar_t *output, const scalar_t *input, index_t classes)
 889: {
 890:   // Each thread block processes a sample in the batch
 891:   input += static_cast<int64_t>(blockIdx.x) * classes;
 892:   output += static_cast<int64_t>(blockIdx.x) * classes;
 893: 
 894:   accscalar_t threadMax = std::numeric_limits<accscalar_t>::lowest();
 895:   accscalar_t threadExp = static_cast<accscalar_t>(0);
 896: 
 897:   // The first smem segment is used to cache input values and the last
 898:   // segment is used for thread block reductions
 899:   extern __shared__ unsigned char smem[];
 900:   auto smem_input_cache = reinterpret_cast<scalar_t*>(smem);
 901:   auto smem_reduction_cache = reinterpret_cast<accscalar_t*>(smem +
 902:     classes * sizeof(scalar_t));
 903: 
 904:   using LoadT = at::native::memory::aligned_vector<scalar_t, ILP>;
 905:   const LoadT* const input_vec_ptr = reinterpret_cast<const LoadT*>(input);
 906:   LoadT* const smem_input_cache_vec_ptr = reinterpret_cast<LoadT*>(smem_input_cache);
```
- EN: This block defines GPU kernel entry point(s) `cunn_SoftMaxForwardSmem`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `cunn_SoftMaxForwardSmem`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 908-909
```cpp
 908:   // Download inputs to shared memory while doing the first step
 909:   // in max calculation
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 910-919
```cpp
 910:   MaxFloat<scalar_t, accscalar_t> maxFunc;
 911:   for (index_t offset = threadIdx.x; offset * ILP < classes; offset += blockDim.x) {
 912:     LoadT crnt_vec = input_vec_ptr[offset];
 913:     smem_input_cache_vec_ptr[offset] = crnt_vec;
 914: 
 915:     #pragma unroll
 916:     for (int i = 0; i < ILP; ++i) {
 917:       threadMax = maxFunc(threadMax, crnt_vec.val[i]);
 918:     }
 919:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 921-922
```cpp
 921:   accscalar_t max_k = blockReduceWarp<Max, accscalar_t>(smem_reduction_cache, threadMax,
 922:     Max<accscalar_t>(), std::numeric_limits<accscalar_t>::lowest());
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 924-925
```cpp
 924:   // Reload input from shared memory to compute the sum. The previous
 925:   // reduce has performed a __syncthreads() so the smem contents are populated.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 926-934
```cpp
 926:   SumExpFloat<scalar_t, accscalar_t> sumExpFunc(max_k);
 927:   for (index_t offset = threadIdx.x; offset * ILP < classes; offset += blockDim.x) {
 928:     LoadT crnt_vec = smem_input_cache_vec_ptr[offset];
 929: 
 930:     #pragma unroll
 931:     for (int i = 0; i < ILP; ++i) {
 932:       threadExp = sumExpFunc(threadExp, crnt_vec.val[i]);
 933:     }
 934:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 936-937
```cpp
 936:   accscalar_t sumAll = blockReduceWarp<Add, accscalar_t>(smem_reduction_cache, threadExp,
 937:     Add<accscalar_t>(), static_cast<accscalar_t>(0));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 939-939
```cpp
 939:   Epilogue<scalar_t, accscalar_t, outscalar_t> epilogue(max_k, sumAll);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 941-941
```cpp
 941:   // Use vectorized stores to save the output
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 942-955
```cpp
 942:   using StoreT = at::native::memory::aligned_vector<outscalar_t, ILP>;
 943:   StoreT* output_vec_ptr = reinterpret_cast<StoreT*>(output);
 944:   for (index_t offset = threadIdx.x; offset * ILP < classes; offset += blockDim.x) {
 945:     LoadT crnt_vec = smem_input_cache_vec_ptr[offset];
 946:     StoreT out_vec;
 947: 
 948:     #pragma unroll
 949:     for (int i = 0; i < ILP; ++i) {
 950:       out_vec.val[i] = epilogue(crnt_vec.val[i]);
 951:     }
 952: 
 953:     output_vec_ptr[offset] = out_vec;
 954:   }
 955: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 957-959
```cpp
 957: C10_DEVICE bool inline is_32bit_representable(const int64_t value) {
 958:   return value < static_cast<int64_t>(std::numeric_limits<int32_t>::max());
 959: }
```
- EN: This block defines or continues the implementation of `is_32bit_representable`.
- CN: 该代码块定义或继续实现 `is_32bit_representable`。

### Lines 961-982
```cpp
 961: template <int ILP, typename scalar_t, typename accscalar_t, typename outscalar_t, template<typename, typename, typename> class Epilogue>
 962: __global__ void
 963: cunn_SoftMaxBackward(scalar_t *gradInput, const outscalar_t *output, const outscalar_t *gradOutput, int64_t classes)
 964: {
 965:   using LoadT = at::native::memory::aligned_vector<scalar_t, ILP>;
 966:   using StoreT = at::native::memory::aligned_vector<outscalar_t, ILP>;
 967: 
 968:   extern __shared__ unsigned char smem[];
 969:   auto sdata = reinterpret_cast<accscalar_t*>(smem);
 970:   gradInput += static_cast<int64_t>(blockIdx.x) * classes;
 971:   output += static_cast<int64_t>(blockIdx.x) * classes;
 972:   gradOutput += static_cast<int64_t>(blockIdx.x) * classes;
 973: 
 974:   const int64_t shift = ((uint64_t)gradInput) % ALIGN_BYTES / sizeof(scalar_t);
 975:   const int64_t output_shift = ((uint64_t)output) % ALIGN_BYTES / sizeof(outscalar_t);
 976:   const int64_t grad_output_shift = ((uint64_t)gradOutput) % ALIGN_BYTES / sizeof(outscalar_t);
 977: 
 978:   const bool can_use_32bit_indexing = is_32bit_representable(shift) && is_32bit_representable(output_shift) && is_32bit_representable(grad_output_shift) && is_32bit_representable(classes);
 979:   accscalar_t threadSum;
 980:   if (can_use_32bit_indexing) {
 981:     threadSum = ilpReduce<AddFloat, ILP, outscalar_t, accscalar_t, int32_t>(
 982:         static_cast<int32_t>(grad_output_shift), gradOutput, classes, AddFloat<outscalar_t, accscalar_t>(), accscalar_t(0));
```
- EN: This block defines GPU kernel entry point(s) `cunn_SoftMaxBackward`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `cunn_SoftMaxBackward`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 983-988
```cpp
 983:   } else {
 984:     threadSum = ilpReduce<AddFloat, ILP, outscalar_t, accscalar_t, int64_t>(
 985:         grad_output_shift, gradOutput, classes, AddFloat<outscalar_t, accscalar_t>(), accscalar_t(0));
 986:   }
 987:   accscalar_t sum_k = blockReduce<Add, accscalar_t>(
 988:         sdata, threadSum, Add<accscalar_t>(), accscalar_t(0));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 990-990
```cpp
 990:   Epilogue<scalar_t, accscalar_t, outscalar_t> epilogue(sum_k);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 992-1005
```cpp
 992:   if (shift == output_shift && shift == grad_output_shift) {
 993:     if (can_use_32bit_indexing) {
 994:       WriteBpropResultsVectorized<ILP, scalar_t, accscalar_t, outscalar_t, Epilogue, int32_t>(classes, static_cast<int32_t>(shift), gradInput, output, gradOutput, epilogue);
 995:     } else {
 996:       WriteBpropResultsVectorized<ILP, scalar_t, accscalar_t, outscalar_t, Epilogue, int64_t>(classes, shift, gradInput, output, gradOutput, epilogue);
 997:     }
 998:   } else {
 999:     if (can_use_32bit_indexing) {
1000:       WriteBpropResults<ILP, scalar_t, accscalar_t, outscalar_t, Epilogue, int32_t>(classes, gradInput, output, gradOutput, epilogue);
1001:     } else {
1002:       WriteBpropResults<ILP, scalar_t, accscalar_t, outscalar_t, Epilogue, int64_t>(classes, gradInput, output, gradOutput, epilogue);
1003:     }
1004:   }
1005: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1007-1028
```cpp
1007: template <int ILP, typename scalar_t, typename accscalar_t, typename outscalar_t, template<typename, typename, typename> class Epilogue>
1008: __global__ void
1009: cunn_SoftMaxBackwardSmem(scalar_t *gradInput, const outscalar_t *output, const outscalar_t *gradOutput, int64_t classes)
1010: {
1011:   // The first smem segment is used to cache input values and the last
1012:   // segment is used for thread block reductions
1013:   extern __shared__ unsigned char smem[];
1014:   auto smem_input_cache = reinterpret_cast<outscalar_t*>(smem);
1015:   auto smem_reduction_cache = reinterpret_cast<accscalar_t*>(smem +
1016:     classes * sizeof(outscalar_t));
1017: 
1018:   gradInput += static_cast<int64_t>(blockIdx.x) * classes;
1019:   output += static_cast<int64_t>(blockIdx.x) * classes;
1020:   gradOutput += static_cast<int64_t>(blockIdx.x) * classes;
1021: 
1022:   accscalar_t threadSum = 0;
1023: 
1024:   using LoadT = at::native::memory::aligned_vector<outscalar_t, ILP>;
1025:   const LoadT* const gradOutput_vec_ptr = reinterpret_cast<const LoadT*>(gradOutput);
1026:   LoadT* const smem_gradOutput_cache_vec_ptr = reinterpret_cast<LoadT*>(smem_input_cache);
1027: 
1028:   // Download inputs to shared memory while doing the first step
```
- EN: This block defines GPU kernel entry point(s) `cunn_SoftMaxBackwardSmem`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `cunn_SoftMaxBackwardSmem`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 1029-1029
```cpp
1029:   // in sum calculation
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1030-1038
```cpp
1030:   for (int32_t offset = threadIdx.x; offset * ILP < classes; offset += blockDim.x) {
1031:     LoadT crnt_vec = gradOutput_vec_ptr[offset];
1032:     smem_gradOutput_cache_vec_ptr[offset] = crnt_vec;
1033: 
1034:     #pragma unroll
1035:     for (int i = 0; i < ILP; ++i) {
1036:       threadSum = threadSum + crnt_vec.val[i];
1037:     }
1038:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1040-1041
```cpp
1040:   // We need a __syncthreads() here to be safe. However, blockReduceWarp's code
1041:   // calls a __syncthreads() before reading shared memory so we are safe.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 1043-1043
```cpp
1043:   accscalar_t sum_k = blockReduceWarp<Add, accscalar_t>(smem_reduction_cache, threadSum, Add<accscalar_t>(), accscalar_t(0));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1045-1045
```cpp
1045:   Epilogue<scalar_t, accscalar_t, outscalar_t> epilogue(sum_k);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1047-1047
```cpp
1047:   // Use vectorized stores to save the output
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1048-1062
```cpp
1048:   using StoreT = at::native::memory::aligned_vector<scalar_t, ILP>;
1049:   StoreT* gradInput_vec_ptr = reinterpret_cast<StoreT*>(gradInput);
1050:   const LoadT* const output_vec_ptr = reinterpret_cast<const LoadT*>(output);
1051:   for (int32_t offset = threadIdx.x; offset * ILP < classes; offset += blockDim.x) {
1052:     LoadT crnt_vec = smem_gradOutput_cache_vec_ptr[offset];
1053:     LoadT crnt_out = output_vec_ptr[offset];
1054:     StoreT out_vec;
1055: 
1056:     #pragma unroll
1057:     for (int i = 0; i < ILP; ++i) {
1058:       out_vec.val[i] = epilogue(crnt_vec.val[i], crnt_out.val[i]);
1059:     }
1060: 
1061:     gradInput_vec_ptr[offset] = out_vec;
1062:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1063-1063
```cpp
1063: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1066-1087
```cpp
1066:  template<template<typename, typename, typename> class Epilogue,
1067:           template<typename, typename, typename> class EpilogueWithMul, bool is_log_softmax, bool use_fast_softmax>
1068: Tensor host_softmax(const Tensor & input_, const int64_t dim_, const bool half_to_float, const Tensor& output){
1069:   if (half_to_float) {
1070:     TORCH_CHECK(input_.scalar_type() == ScalarType::Half, "conversion is supported for Half type only");
1071:   }
1072:   auto input = input_.contiguous();
1073:   static_assert(std::is_same_v<acc_type<at::Half, true>, float>, "accscalar_t for half should be float");
1074:   if (input.dim() == 0) input = input.view(1);
1075:   int64_t dim = maybe_wrap_dim(dim_, input.dim());
1076:   TORCH_CHECK(dim >=0 && dim < input.dim(), "dim must be non-negative and less than input dimensions");
1077:   int64_t outer_size = 1;
1078:   int64_t dim_size = input.size(dim);
1079: 
1080:   if (input.numel() > 0) {
1081:     int64_t inner_size = 1;
1082:     cudaStream_t stream = at::cuda::getCurrentCUDAStream();
1083:     for (int64_t i = 0; i < dim; ++i)
1084:       outer_size *= input.size(i);
1085:     for (int64_t i = dim + 1; i < input.dim(); ++i)
1086:       inner_size *= input.size(i);
1087:     // This kernel spawns a block per each element in the batch.
```
- EN: This block defines or continues the implementation of `host_softmax`.
- CN: 该代码块定义或继续实现 `host_softmax`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1088-1088
```cpp
1088:     // XXX: it assumes that inner_size == 1
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1090-1111
```cpp
1090:     if (inner_size == 1) {
1091:       dim3 grid(outer_size);
1092:       AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, input.scalar_type(), "host_softmax", [&] {
1093:         using accscalar_t = acc_type<scalar_t, true>;
1094:         if (!half_to_float) {
1095:           auto output_ptr = output.mutable_data_ptr<scalar_t>();
1096:           auto input_ptr = input.const_data_ptr<scalar_t>();
1097:           if (dim_size <= 2048 && dim_size*sizeof(scalar_t) <= 8192) {
1098:             int64_t remaining = outer_size;
1099:             int64_t chunk_size = (1L << 30L) / dim_size;
1100:             while(remaining > 0) {
1101:               dispatch_softmax_forward<scalar_t, scalar_t, accscalar_t, is_log_softmax, false>(
1102:                 output_ptr, input_ptr, dim_size, dim_size, std::min<int64_t>(remaining, chunk_size), nullptr/* not masked */);
1103:               input_ptr += chunk_size * dim_size;
1104:               output_ptr += chunk_size * dim_size;
1105:               remaining -= chunk_size;
1106:             }
1107:           } else {
1108:             constexpr int ILP = sizeof(float4) / sizeof(scalar_t);
1109:             if constexpr (use_fast_softmax) {
1110:               dim3 block(512);
1111:               size_t smem_reduction_sz = block.x / at::cuda::warp_size() * sizeof(accscalar_t);
```
- EN: This block defines or continues the implementation of `constexpr`.
- CN: 该代码块定义或继续实现 `constexpr`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1112-1123
```cpp
1112:               if (dim_size % ILP == 0) {
1113:                 cunn_SoftMaxForwardGmem<ILP, scalar_t, accscalar_t, scalar_t, EpilogueWithMul>
1114:                     <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1115:               } else {
1116:                 cunn_SoftMaxForwardFast<ILP, scalar_t, accscalar_t, scalar_t, EpilogueWithMul>
1117:                     <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1118:               }
1119:             } else {
1120:               dim3 block = SoftMaxForward_getBlockSize(dim_size);
1121:               size_t smem_reduction_sz = block.x / at::cuda::warp_size() * sizeof(accscalar_t);
1122:               auto max_elements_per_smem = (at::cuda::getCurrentDeviceProperties()->sharedMemPerBlock -
1123:                 smem_reduction_sz) / sizeof(scalar_t);
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 1125-1128
```cpp
1125:               bool can_use_smem = static_cast<size_t>(dim_size) < max_elements_per_smem;
1126:               can_use_smem &= !(reinterpret_cast<uintptr_t>(input_ptr) % ALIGN_BYTES);
1127:               can_use_smem &= (!(reinterpret_cast<uintptr_t>(output_ptr) % ALIGN_BYTES));
1128:               can_use_smem &= !(dim_size % ILP);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1130-1151
```cpp
1130:               int32_t potential_reg_cnt = potential_register_count(dim_size, block.x);
1131:               if(potential_reg_cnt < 10){
1132:                 TORCH_INTERNAL_ASSERT(potential_reg_cnt > 0, "potential_reg_cnt for softmax with register should be greater than 0.");
1133:                 switch (potential_reg_cnt) {
1134:                   // TODO(Wenqin): try to investigate why we couldn't use macro for below code,
1135:                   // because it seems on MSVS, it seems the macro way didn't expand correct.
1136:                   case 1:
1137:                     cunn_SoftMaxForwardReg<scalar_t, accscalar_t, scalar_t, Epilogue, int64_t, 1>
1138:                       <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1139:                     break;
1140:                   case 2:
1141:                     cunn_SoftMaxForwardReg<scalar_t, accscalar_t, scalar_t, Epilogue, int64_t, 2>
1142:                       <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1143:                     break;
1144:                   case 3:
1145:                     cunn_SoftMaxForwardReg<scalar_t, accscalar_t, scalar_t, Epilogue, int64_t, 3>
1146:                       <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1147:                     break;
1148:                   case 4:
1149:                     cunn_SoftMaxForwardReg<scalar_t, accscalar_t, scalar_t, Epilogue, int64_t, 4>
1150:                       <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1151:                     break;
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1152-1165
```cpp
1152:                   case 5:
1153:                     cunn_SoftMaxForwardReg<scalar_t, accscalar_t, scalar_t, Epilogue, int64_t, 5>
1154:                       <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1155:                     break;
1156:                   case 6:
1157:                     cunn_SoftMaxForwardReg<scalar_t, accscalar_t, scalar_t, Epilogue, int64_t, 6>
1158:                       <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1159:                     break;
1160:                   case 7:
1161:                     cunn_SoftMaxForwardReg<scalar_t, accscalar_t, scalar_t, Epilogue, int64_t, 7>
1162:                       <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1163:                     break;
1164:                   case 8:
1165:                     cunn_SoftMaxForwardReg<scalar_t, accscalar_t, scalar_t, Epilogue, int64_t, 8>
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1166-1179
```cpp
1166:                       <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1167:                     break;
1168:                   case 9:
1169:                     cunn_SoftMaxForwardReg<scalar_t, accscalar_t, scalar_t, Epilogue, int64_t, 9>
1170:                       <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1171:                     break;
1172:                 }
1173:               } else if (can_use_smem) {
1174:                 size_t smem_sz = dim_size * sizeof(scalar_t) + smem_reduction_sz;
1175:                 cunn_SoftMaxForwardSmem<ILP, scalar_t, accscalar_t, scalar_t, Epilogue>
1176:                   <<<grid, block, smem_sz, stream>>>(output_ptr, input_ptr, dim_size);
1177:               } else {
1178:                 cunn_SoftMaxForward<ILP, scalar_t, accscalar_t, scalar_t, Epilogue>
1179:                   <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1180-1181
```cpp
1180:               }
1181:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1183-1197
```cpp
1183:             C10_CUDA_KERNEL_LAUNCH_CHECK();
1184:           }
1185:         } else {
1186:           auto output_ptr = output.mutable_data_ptr<accscalar_t>();
1187:           auto input_ptr = input.const_data_ptr<scalar_t>();
1188:           if (dim_size <= 1024 && dim_size*sizeof(scalar_t) <= 4096) {
1189:             int64_t remaining = outer_size;
1190:             int64_t chunk_size = (1<<30) / dim_size;
1191:             while(remaining > 0) {
1192:               dispatch_softmax_forward<scalar_t, accscalar_t, accscalar_t, is_log_softmax, false>(
1193:                   output_ptr, input_ptr, dim_size, dim_size, std::min<int64_t>(remaining, chunk_size), nullptr/* not masked */);
1194:               input_ptr += chunk_size * dim_size;
1195:               output_ptr += chunk_size * dim_size;
1196:               remaining -= chunk_size;
1197:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1198-1219
```cpp
1198:           } else {
1199:             constexpr int ILP = sizeof(float4) / sizeof(scalar_t);
1200:             if constexpr (use_fast_softmax) {
1201:               dim3 block(512);
1202:               size_t smem_reduction_sz = block.x / at::cuda::warp_size() * sizeof(accscalar_t);
1203:               if (dim_size % ILP == 0) {
1204:                 cunn_SoftMaxForwardGmem<ILP, scalar_t, accscalar_t, accscalar_t, EpilogueWithMul>
1205:                     <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1206:               } else {
1207:                 cunn_SoftMaxForwardFast<ILP, scalar_t, accscalar_t, accscalar_t, EpilogueWithMul>
1208:                     <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1209:               }
1210:             } else {
1211:               dim3 block = SoftMaxForward_getBlockSize(dim_size);
1212:               size_t smem_reduction_sz = block.x / at::cuda::warp_size() * sizeof(accscalar_t);
1213:               auto max_elements_per_smem = (at::cuda::getCurrentDeviceProperties()->sharedMemPerBlock -
1214:                 smem_reduction_sz) / sizeof(scalar_t);
1215: 
1216:               bool can_use_smem = static_cast<size_t>(dim_size) < max_elements_per_smem;
1217:               can_use_smem &= !(reinterpret_cast<uintptr_t>(input_ptr) % ALIGN_BYTES);
1218:               can_use_smem &= (!(reinterpret_cast<uintptr_t>(output_ptr) % ALIGN_BYTES));
1219:               can_use_smem &= !(dim_size % ILP);
```
- EN: This block defines or continues the implementation of `constexpr`.
- CN: 该代码块定义或继续实现 `constexpr`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1221-1229
```cpp
1221:               if (can_use_smem) {
1222:                 size_t smem_sz = dim_size * sizeof(scalar_t) + smem_reduction_sz;
1223:                 cunn_SoftMaxForwardSmem<ILP, scalar_t, accscalar_t, accscalar_t, Epilogue>
1224:                   <<<grid, block, smem_sz, stream>>>(output_ptr, input_ptr, dim_size);
1225:               } else {
1226:                 cunn_SoftMaxForward<ILP, scalar_t, accscalar_t, accscalar_t, Epilogue>
1227:                   <<<grid, block, smem_reduction_sz, stream>>>(output_ptr, input_ptr, dim_size);
1228:               }
1229:             }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1231-1244
```cpp
1231:             C10_CUDA_KERNEL_LAUNCH_CHECK();
1232:           }
1233:         }
1234:       });
1235:     // This kernel runs in a 2D grid, where each application along y dimension has a fixed
1236:     // outer_size, and runs in parallel over inner_size. Dimension x is parallel over outer_size.
1237:     // Reductions over dim are done in a single-threaded manner.
1238:     } else {
1239:       uint32_t smem_size;
1240:       dim3 grid, block;
1241:       AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, input.scalar_type(), "host_softmax", [&] {
1242:         using accscalar_t = acc_type<scalar_t, true>;
1243:         AT_DISPATCH_INDEX_TYPES(
1244:             at::native::canUse32BitIndexMath(input, INT_MAX) ? ScalarType::Int : ScalarType::Long,
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 1245-1265
```cpp
1245:         "host_softmax_launcher", [&] {
1246:             if (!half_to_float) {
1247:                 SpatialSoftMax_getLaunchSizes<accscalar_t>(
1248:                     &cunn_SpatialSoftMaxForward<scalar_t, accscalar_t, scalar_t, index_t, Epilogue>,
1249:                     outer_size, dim_size, inner_size,
1250:                     grid, block, smem_size);
1251:                 cunn_SpatialSoftMaxForward<scalar_t, accscalar_t, scalar_t, index_t, Epilogue>
1252:                   <<<grid, block, smem_size, stream>>>(
1253:                   output.mutable_data_ptr<scalar_t>(), input.const_data_ptr<scalar_t>(), outer_size, dim_size, inner_size);
1254:                 C10_CUDA_KERNEL_LAUNCH_CHECK();
1255:             } else {
1256:                 SpatialSoftMax_getLaunchSizes<accscalar_t>(
1257:                     &cunn_SpatialSoftMaxForward<scalar_t, accscalar_t, accscalar_t, index_t, Epilogue>,
1258:                     outer_size, dim_size, inner_size,
1259:                     grid, block, smem_size);
1260:                 cunn_SpatialSoftMaxForward<scalar_t, accscalar_t, accscalar_t, index_t, Epilogue>
1261:                   <<<grid, block, smem_size, stream>>>(
1262:                   output.mutable_data_ptr<accscalar_t>(), input.const_data_ptr<scalar_t>(), outer_size, dim_size, inner_size);
1263:                 C10_CUDA_KERNEL_LAUNCH_CHECK();
1264:             }
1265:          });
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1266-1270
```cpp
1266:       });
1267:     }
1268:   }
1269:   return output;
1270: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1272-1293
```cpp
1272: template<typename input_t, typename output_t, typename accscalar_t, template<typename, typename, typename> class Epilogue>
1273: void dispatch_host_softmax_backward(int64_t dim_size, dim3 grid, Tensor &grad, Tensor &output, const Tensor &gI) {
1274:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
1275:   constexpr int ILP = sizeof(float4) / sizeof(output_t);
1276:   dim3 block = SoftMax_getBlockSize(ILP, dim_size);
1277: 
1278:   size_t smem_reduction_sz = block.x / at::cuda::warp_size() * sizeof(accscalar_t);
1279:   auto max_elements_per_smem = (at::cuda::getCurrentDeviceProperties()->sharedMemPerBlock -
1280:     smem_reduction_sz) / sizeof(output_t);
1281:   bool can_use_smem = static_cast<size_t>(dim_size) < max_elements_per_smem;
1282:   can_use_smem &= (!(reinterpret_cast<uintptr_t>(gI.const_data_ptr<input_t>()) % ALIGN_BYTES));
1283:   can_use_smem &= (!(reinterpret_cast<uintptr_t>(output.const_data_ptr<output_t>()) % ALIGN_BYTES));
1284:   can_use_smem &= !(reinterpret_cast<uintptr_t>(grad.const_data_ptr<output_t>()) % ALIGN_BYTES);
1285:   can_use_smem &= !(dim_size % ILP);
1286:   // This should not be needed on current generation GPUs because the size of shared memory is so low.
1287:   // But we add this check to be defensive and future-proof just in case shared memory size goes up
1288:   // to be so large as to requires 64-bits of addressing.
1289:   can_use_smem &= (dim_size < std::numeric_limits<int32_t>::max());
1290: 
1291:   if (can_use_smem) {
1292:     size_t smem_sz = dim_size * sizeof(output_t) + smem_reduction_sz;
1293:     cunn_SoftMaxBackwardSmem<ILP, input_t, accscalar_t, output_t, Epilogue>
```
- EN: This block defines or continues the implementation of `dispatch_host_softmax_backward`.
- CN: 该代码块定义或继续实现 `dispatch_host_softmax_backward`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 1294-1303
```cpp
1294:     <<<grid, block, smem_sz, stream>>>(
1295:       gI.mutable_data_ptr<input_t>(), output.const_data_ptr<output_t>(), grad.const_data_ptr<output_t>(), dim_size);
1296:   } else {
1297:     cunn_SoftMaxBackward<ILP, input_t, accscalar_t, output_t, Epilogue>
1298:     <<<grid, block, block.x * sizeof(accscalar_t), stream>>>(
1299:         gI.mutable_data_ptr<input_t>(), output.const_data_ptr<output_t>(), grad.const_data_ptr<output_t>(), dim_size
1300:       );
1301:   }
1302:   C10_CUDA_KERNEL_LAUNCH_CHECK();
1303: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1305-1326
```cpp
1305: template<template<typename, typename, typename> class Epilogue, bool is_log_softmax>
1306: void host_softmax_backward(const Tensor &grad_, const Tensor &output_, int64_t dim_, bool half_to_float, const Tensor &gI){
1307:   int64_t dim = maybe_wrap_dim(dim_, grad_.dim());
1308:   if (grad_.numel() == 0) {
1309:     return;
1310:   }
1311:   auto grad = grad_.contiguous();
1312:   static_assert(std::is_same_v<acc_type<at::Half, true>, float>, "accscalar_t for half should be float");
1313:   if (grad.dim() == 0) grad = grad.view(1);
1314:   TORCH_CHECK(dim >=0 && dim < grad.dim(), "dim must be non-negative and less than input dimensions");
1315:   auto output = output_.contiguous();
1316:   if (output.dim() == 0) output = output.view(1);
1317:   int64_t outer_size = 1;
1318:   int64_t dim_size = output.size(dim);
1319:   int64_t inner_size = 1;
1320:   for (int64_t i = 0; i < dim; ++i)
1321:     outer_size *= output.size(i);
1322:   for (int64_t i = dim + 1; i < output.dim(); ++i)
1323:     inner_size *= output.size(i);
1324: // See descriptions of kernels above.
1325:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
1326:   if (inner_size == 1) {
```
- EN: This block defines or continues the implementation of `host_softmax_backward`.
- CN: 该代码块定义或继续实现 `host_softmax_backward`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1327-1348
```cpp
1327:     dim3 grid(outer_size);
1328:     AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, gI.scalar_type(), "host_softmax_backward", [&] {
1329:     using accscalar_t = acc_type<scalar_t, true>;
1330:     if (!half_to_float) {
1331:       if (dim_size <= 1024 && dim_size*sizeof(scalar_t) <= 4096) {
1332:         auto gI_ptr = gI.mutable_data_ptr<scalar_t>();
1333:         auto grad_ptr = grad.const_data_ptr<scalar_t>();
1334:         auto output_ptr = output.const_data_ptr<scalar_t>();
1335:         int64_t remaining = outer_size;
1336:         int64_t chunk_size = (1<<30) / dim_size;
1337:         while(remaining > 0) {
1338:           dispatch_softmax_backward<scalar_t, scalar_t, accscalar_t, is_log_softmax, false /* masked_softmax */>(
1339:             gI_ptr, grad_ptr, output_ptr, dim_size, dim_size, std::min<int64_t>(remaining, chunk_size));
1340:           gI_ptr += chunk_size * dim_size;
1341:           grad_ptr += chunk_size * dim_size;
1342:           output_ptr += chunk_size * dim_size;
1343:           remaining -= chunk_size;
1344:         }
1345:       } else {
1346:         dispatch_host_softmax_backward<scalar_t, scalar_t, accscalar_t, Epilogue>(dim_size, grid, grad, output, gI);
1347:       }
1348:     } else {
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 1349-1365
```cpp
1349:       if (dim_size <= 1024 && dim_size*sizeof(scalar_t) <= 4096) {
1350:         auto gI_ptr = gI.mutable_data_ptr<scalar_t>();
1351:         auto grad_ptr = grad.const_data_ptr<accscalar_t>();
1352:         auto output_ptr = output.const_data_ptr<accscalar_t>();
1353:         int64_t remaining = outer_size;
1354:         int64_t chunk_size = (1<<30) / dim_size;
1355:         while(remaining > 0) {
1356:           dispatch_softmax_backward<accscalar_t, scalar_t, accscalar_t, is_log_softmax, false /* masked_softmax */>(
1357:             gI_ptr, grad_ptr, output_ptr, dim_size, dim_size, std::min<int64_t>(remaining, chunk_size));
1358:           gI_ptr += chunk_size * dim_size;
1359:           grad_ptr += chunk_size * dim_size;
1360:           output_ptr += chunk_size * dim_size;
1361:           remaining -= chunk_size;
1362:         }
1363:       } else {
1364:         dispatch_host_softmax_backward<scalar_t, accscalar_t, accscalar_t, Epilogue>(dim_size, grid, grad, output, gI);
1365:       }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1366-1377
```cpp
1366:     }
1367:     });
1368:   } else {
1369:     uint32_t smem_size;
1370:     dim3 grid, block;
1371:     AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, gI.scalar_type(), "host_softmax_backward", [&] {
1372:       using accscalar_t = acc_type<scalar_t, true>;
1373:       if (!half_to_float) {
1374:           SpatialSoftMax_getLaunchSizes<accscalar_t>(
1375:               &cunn_SpatialSoftMaxBackward<scalar_t, accscalar_t, scalar_t, Epilogue>,
1376:               outer_size, dim_size, inner_size,
1377:               grid, block, smem_size);
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 1379-1389
```cpp
1379:           cunn_SpatialSoftMaxBackward<scalar_t, accscalar_t, scalar_t, Epilogue>
1380:             <<<grid, block, smem_size, stream>>>(
1381:               gI.mutable_data_ptr<scalar_t>(), output.const_data_ptr<scalar_t>(), grad.const_data_ptr<scalar_t>(),
1382:               outer_size, dim_size, inner_size
1383:           );
1384:           C10_CUDA_KERNEL_LAUNCH_CHECK();
1385:       } else {
1386:           SpatialSoftMax_getLaunchSizes<accscalar_t>(
1387:               &cunn_SpatialSoftMaxBackward<scalar_t, accscalar_t, accscalar_t, Epilogue>,
1388:               outer_size, dim_size, inner_size,
1389:               grid, block, smem_size);
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1391-1401
```cpp
1391:           cunn_SpatialSoftMaxBackward<scalar_t, accscalar_t, accscalar_t, Epilogue>
1392:             <<<grid, block, smem_size, stream>>>(
1393:               gI.mutable_data_ptr<scalar_t>(), output.const_data_ptr<accscalar_t>(), grad.const_data_ptr<accscalar_t>(),
1394:               outer_size, dim_size, inner_size
1395:           );
1396:           C10_CUDA_KERNEL_LAUNCH_CHECK();
1397:       }
1398:     });
1399:   }
1400: }
1401: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1403-1409
```cpp
1403: TORCH_IMPL_FUNC(log_softmax_cuda_out) (
1404:   const Tensor &input,
1405:   const int64_t dim,
1406:   const bool half_to_float,
1407:   const Tensor &output) {
1408:   host_softmax<LogSoftMaxForwardEpilogue, LogSoftMaxForwardEpilogue, true, false>(input, dim, half_to_float, output);
1409: }
```
- EN: This block defines or continues the implementation of `log_softmax_cuda_out`.
- CN: 该代码块定义或继续实现 `log_softmax_cuda_out`。

### Lines 1411-1425
```cpp
1411: TORCH_IMPL_FUNC(log_softmax_backward_cuda_out) (
1412:   const Tensor& grad,
1413:   const Tensor& output,
1414:   int64_t dim,
1415:   ScalarType input_dtype,
1416:   const Tensor& grad_input) {
1417:   bool half_to_float = grad.scalar_type() != input_dtype;
1418:   if (half_to_float) {
1419:     TORCH_CHECK(
1420:         (grad.scalar_type() == ScalarType::Float &&
1421:          input_dtype == ScalarType::Half),
1422:         "expected input and grad types to match, or input to be at::Half and grad to be at::Float");
1423:   }
1424:   host_softmax_backward<LogSoftMaxBackwardEpilogue, true>(grad, output, dim, half_to_float, grad_input);
1425: }
```
- EN: This block defines or continues the implementation of `log_softmax_backward_cuda_out`.
- CN: 该代码块定义或继续实现 `log_softmax_backward_cuda_out`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1427-1437
```cpp
1427: TORCH_IMPL_FUNC(softmax_cuda_out) (
1428:   const Tensor &input,
1429:   const int64_t dim,
1430:   const bool half_to_float,
1431:   const Tensor &output) {
1432: #if defined(USE_ROCM)
1433:    host_softmax<SoftMaxForwardEpilogue, SoftMaxForwardWithMulEpilogue, false, true>(input, dim, half_to_float, output);
1434:  #else
1435:    host_softmax<SoftMaxForwardEpilogue, SoftMaxForwardWithMulEpilogue, false, false>(input, dim, half_to_float, output);
1436:  #endif
1437: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `softmax_cuda_out`.
- CN: 该代码块定义或继续实现 `softmax_cuda_out`。

### Lines 1439-1454
```cpp
1439: TORCH_IMPL_FUNC(softmax_backward_cuda_out)
1440: (const Tensor& grad,
1441:  const Tensor& output,
1442:  int64_t dim,
1443:  ScalarType input_dtype,
1444:  const Tensor& grad_input) {
1445:   bool half_to_float = grad.scalar_type() != input_dtype;
1446:   if (half_to_float) {
1447:     TORCH_CHECK(
1448:         (grad.scalar_type() == ScalarType::Float &&
1449:          input_dtype == ScalarType::Half),
1450:         "expected input and grad types to match, or input to be at::Half and grad to be at::Float");
1451:   }
1452:   Tensor tmp = grad * output;
1453:   host_softmax_backward<SoftMaxBackwardEpilogue, false>(tmp, output, dim, half_to_float, grad_input);
1454: }
```
- EN: This block defines or continues the implementation of `softmax_backward_cuda_out`.
- CN: 该代码块定义或继续实现 `softmax_backward_cuda_out`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1456-1477
```cpp
1456: Tensor masked_softmax_cuda(const Tensor& input_, const Tensor& mask_, const std::optional<int64_t> dim_, const std::optional<int64_t> mask_type_) {
1457:   Tensor output = at::empty_like(input_, input_.options());
1458:   TORCH_CHECK(mask_.scalar_type() == ScalarType::Bool, "Mask should be a boolean tensor");
1459: 
1460:   TORCH_CHECK(mask_type_.has_value(), "Mask Type should be defined");
1461:   int64_t mask_type = mask_type_.value();
1462:   TORCH_CHECK((mask_type == 0) || (mask_type == 1) || (mask_type == 2), "Mask Type should be 0 (src_mask), 1 (src_key_padding_mask), or 2 (default_mask)");
1463: 
1464:   // If input is [B, H, T, T] and mask is [B, T]
1465:   // we have special fast kernel
1466:   // mask_type == 1 => mask_ is a src_key_padding_mask
1467:   bool is_BxT_mask = (mask_type == 1) && (input_.dim() == 4 && mask_.dim() == 2 && input_.size(0) == mask_.size(0) && input_.size(2) == mask_.size(1) && input_.size(3) == mask_.size(1));
1468: 
1469:   // If input is [B, H, T, T] and mask is [T, T]
1470:   // expand mask to [B, H, T, T] and treat it like regular mask
1471:   // TODO We should have special fast kernel for TxT mask as well
1472:   // mask_type == 0 => mask_ is a src_mask
1473:   bool is_TxT_mask = (mask_type == 0) && input_.dim() == 4 && mask_.dim() == 2 && input_.size(3) == mask_.size(1) && input_.size(2) == mask_.size(0) && mask_.size(0) == mask_.size(1);
1474:   // If mask_type == 2, then mask_.sizes() must equal input_.sizes()
1475:   TORCH_CHECK(mask_.sizes() == input_.sizes() || is_BxT_mask || is_TxT_mask, "Mask shape should match input. mask: ", mask_.sizes(), " input: ", input_.sizes());
1476: 
1477:   auto input = input_.dim() == 0 ? input_.view(1) : input_;
```
- EN: This block defines or continues the implementation of `masked_softmax_cuda`.
- CN: 该代码块定义或继续实现 `masked_softmax_cuda`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1478-1482
```cpp
1478:   auto mask = mask_.dim() == 0 ? mask_.view(1) : mask_;
1479:   if (is_TxT_mask) {
1480:     mask = mask.expand(input.sizes());
1481:   }
1482:   int64_t dim = dim_.has_value() ? dim_.value() : input.dim() - 1;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1484-1504
```cpp
1484:   int softmax_elements = input.size(dim);
1485:   // Persistent softmax is only supported when all of the conditions are held:
1486:   //     1) softmax_elements <= 1024
1487:   //     2) softmax_elements * input.element_size() <= 4096
1488:   //     3) mask.is_contiguous()
1489:   //     4) dim == input.dim() - 1
1490:   // Otherwise, we fallback to vanilla softmax (where we do not support transformer_mask since converting the mask is expensive)
1491:   if (softmax_elements > 1024 || softmax_elements * input.element_size() > 4096 || !mask.is_contiguous() || dim < input.dim()-1) {
1492:     if (is_BxT_mask) {
1493:       mask = mask.view({mask_.size(0), 1, 1, mask_.size(1)}).expand(input.sizes());
1494:     }
1495:     AT_DISPATCH_FLOATING_TYPES_AND2(
1496:       ScalarType::Half,
1497:       ScalarType::BFloat16,
1498:       input.scalar_type(),
1499:       "masked_softmax",
1500:       [&] {
1501:         output = at::softmax(input.masked_fill(mask, -std::numeric_limits<scalar_t>::infinity()), dim);
1502:       });
1503:     return output;
1504:   }
```
- EN: This block defines or continues the implementation of `element_size`.
- CN: 该代码块定义或继续实现 `element_size`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1505-1526
```cpp
1505:   int batch_count = input.numel() / softmax_elements;
1506:   int chunk_size = input.numel() / input.size(0);
1507:   if (is_BxT_mask) {
1508:     // Only support when num_heads is even in transformer
1509:     TORCH_CHECK(input.size(1) % 2 == 0, "Only support when num_heads is even in transformer");
1510:     AT_DISPATCH_FLOATING_TYPES_AND2(
1511:       ScalarType::Half,
1512:       ScalarType::BFloat16,
1513:       input.scalar_type(),
1514:       "masked_softmax",
1515:       [&] {
1516:         using accscalar_t = acc_type<scalar_t, true>;
1517:         dispatch_softmax_forward<scalar_t, scalar_t, accscalar_t, false/* is_log_softmax */, true/* is_masked */>(
1518:           output.mutable_data_ptr<scalar_t>(),    // dst
1519:           input.const_data_ptr<scalar_t>(),       // src
1520:           softmax_elements,
1521:           softmax_elements,
1522:           batch_count,
1523:           mask.const_data_ptr<bool>(),
1524:           chunk_size,
1525:           true // is_transformer_mask
1526:         );
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1527-1527
```cpp
1527:       });
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1529-1545
```cpp
1529:   } else {
1530:     AT_DISPATCH_FLOATING_TYPES_AND2(
1531:       ScalarType::Half,
1532:       ScalarType::BFloat16,
1533:       input.scalar_type(),
1534:       "masked_softmax",
1535:       [&] {
1536:         using accscalar_t = acc_type<scalar_t, true>;
1537:         dispatch_softmax_forward<scalar_t, scalar_t, accscalar_t, false/* is_log_softmax */, true/* is_masked */>(
1538:           output.mutable_data_ptr<scalar_t>(),    // dst
1539:           input.const_data_ptr<scalar_t>(),       // src
1540:           softmax_elements,
1541:           softmax_elements,
1542:           batch_count,
1543:           mask.const_data_ptr<bool>()
1544:         );
1545:       });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 1546-1548
```cpp
1546:   }
1547:   return output;
1548: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1550-1571
```cpp
1550: Tensor masked_softmax_backward_cuda(
1551:     const Tensor& grad_,
1552:     const Tensor& output_,
1553:     const Tensor& mask_,
1554:     const std::optional<int64_t> dim_) {
1555:   Tensor grad_input = at::empty_like(grad_, grad_.options());
1556:   if (grad_.numel() == 0) {
1557:     return grad_input;
1558:   }
1559: 
1560:   auto grad = grad_.contiguous();
1561:   auto output = output_.contiguous();
1562:   auto mask = mask_.contiguous();
1563:   int64_t dim = dim_.has_value() ? maybe_wrap_dim(dim_.value(), output.dim()) : output.dim() - 1;
1564: 
1565:   grad = grad.dim() == 0 ? grad.view(1) : grad;
1566:   mask = mask.dim() == 0 ? mask.view(1) : mask;
1567:   output = output.dim() == 0 ? output.view(1) : output;
1568: 
1569:   TORCH_CHECK(dim >=0 && dim < grad.dim(), "dim must be non-negative and less than input dimensions");
1570:   TORCH_CHECK(grad.sizes() == mask.sizes(), "Mask shape should match grad shape");
1571:   TORCH_CHECK(mask.scalar_type() == ScalarType::Bool, "Mask should be a boolean tensor");
```
- EN: This block defines or continues the implementation of `masked_softmax_backward_cuda`.
- CN: 该代码块定义或继续实现 `masked_softmax_backward_cuda`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1573-1574
```cpp
1573:   int softmax_elements = output.size(dim);
1574:   int64_t batch_count = grad.numel() / softmax_elements;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1576-1597
```cpp
1576:   if (softmax_elements > 1024 || softmax_elements * grad.element_size() > 4096 || dim < grad.dim()-1) {
1577:     AT_DISPATCH_FLOATING_TYPES_AND2(
1578:       ScalarType::Half,
1579:       ScalarType::BFloat16,
1580:       grad_input.scalar_type(),
1581:       "masked_softmax_backward",
1582:       [&] {
1583:         grad_input = at::_softmax_backward_data(
1584:           grad,
1585:           output.masked_fill(mask, 0),
1586:           dim,
1587:           grad.scalar_type()
1588:         );
1589:       });
1590:   } else {
1591:     grad = grad * output;
1592:     AT_DISPATCH_FLOATING_TYPES_AND2(
1593:       ScalarType::Half,
1594:       ScalarType::BFloat16,
1595:       grad_input.scalar_type(),
1596:       "masked_softmax_backward",
1597:       [&] {
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1598-1611
```cpp
1598:         using accscalar_t = acc_type<scalar_t, true>;
1599:         dispatch_softmax_backward<scalar_t, scalar_t, accscalar_t, false, true /* masked_softmax */>(
1600:           grad_input.mutable_data_ptr<scalar_t>(),  // gI_ptr
1601:           grad.const_data_ptr<scalar_t>(),  // grad_ptr
1602:           output.const_data_ptr<scalar_t>(),  // output_ptr
1603:           softmax_elements,  // softmax_elements
1604:           softmax_elements,   // softmax_elements_stride
1605:           batch_count,  // batch_count
1606:           mask.const_data_ptr<bool>()  /* not masked */
1607:         );
1608:       });
1609:   }
1610:   return grad_input;
1611: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1613-1613
```cpp
1613: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_IMPL_FUNC` provides an out/inplace-style structured kernel implementation. / `TORCH_IMPL_FUNC` 提供结构化的 out/inplace 风格内核实现。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/TensorUtils.h>`
  - `<ATen/TensorOperators.h>`
  - `<ATen/WrapDimUtils.h>`
  - `<c10/macros/Macros.h>`
  - `<ATen/AccumulateType.h>`
  - `<ATen/cuda/NumericLimits.cuh>`
  - `<type_traits>`
  - `<ATen/native/cuda/Loops.cuh>`
  - `<ATen/native/cuda/MemoryAccess.cuh>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `AT_DISPATCH_INDEX_TYPES`
  - `TORCH_IMPL_FUNC`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::warp_size`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
