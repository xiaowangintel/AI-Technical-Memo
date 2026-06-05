# MultinomialKernel.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/MultinomialKernel.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `_isinf`, `renormRows`, `binarySearchForMultinomial`, `sampleMultinomialWithReplacement`.
- 用途（中文）: 实现与 `_isinf`, `renormRows`, `binarySearchForMultinomial`, `sampleMultinomialWithReplacement` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/AccumulateType.h>
   4: #include <ATen/ceil_div.h>
   5: #include <ATen/Dispatch.h>
   6: #include <ATen/Utils.h>
   7: #include <ATen/cuda/CUDAContext.h>
   8: #include <ATen/cuda/EmptyTensor.h>
   9: #include <ATen/cuda/detail/KernelUtils.h>
  10: #include <ATen/native/UnaryOps.h>
  11: #include <ATen/native/cuda/LaunchUtils.h>
  12: #include <ATen/cuda/CUDAGraphsUtils.cuh>
  13: #include <ATen/native/cuda/block_reduce.cuh>
  14: 
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/ceil_div.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/ceil_div.h>`。

### Lines 15-28
```cpp
  15: #ifndef AT_PER_OPERATOR_HEADERS
  16: #include <ATen/CUDAFunctions.h>
  17: #include <ATen/NativeFunctions.h>
  18: #else
  19: #include <ATen/ops/empty_native.h>
  20: #include <ATen/ops/empty_like_native.h>
  21: #include <ATen/ops/cumsum_cuda_dispatch.h>
  22: #include <ATen/ops/uniform_native.h>
  23: #endif
  24: 
  25: #include <curand.h>
  26: #include <curand_kernel.h>
  27: #include <curand_philox4x32_x.h>
  28: #include <type_traits>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/CUDAFunctions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/empty_native.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/CUDAFunctions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/empty_native.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 30-51
```cpp
  30: namespace at::native {
  31: 
  32: namespace {
  33: 
  34: template <
  35:     typename T,
  36:     typename = std::enable_if_t<
  37:         std::is_floating_point_v<T> || std::is_convertible_v<T, float>>>
  38: inline __device__ bool _isinf(T x) {
  39:   if constexpr (std::is_floating_point_v<T>) {
  40:     return ::isinf(x);
  41:   } else {
  42:     return ::isinf(static_cast<float>(x));
  43:   }
  44: }
  45: 
  46: #define MAX_NUM_BLOCKS 200
  47: 
  48: // Normalizes the L1 norm of every row to 1; used by multinomial
  49: template <typename scalar_t>
  50: C10_LAUNCH_BOUNDS_1(cuda::detail::CUDA_NUM_THREADS)
  51: __global__ void renormRowsL1(scalar_t* dist, long rows, long cols) {
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `renormRowsL1`, `_isinf`, `constexpr`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `renormRowsL1`, `_isinf`, `constexpr`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 52-73
```cpp
  52:   extern __shared__  unsigned char my_smem[];
  53:   scalar_t *smem = reinterpret_cast<scalar_t *>(my_smem);
  54:   scalar_t zero = static_cast<scalar_t>(0);
  55:   scalar_t val;
  56:   for (int64_t row = blockIdx.x; row < rows; row += gridDim.x) {
  57:     scalar_t sum = static_cast<scalar_t>(0);
  58:     for (int64_t col = threadIdx.x; col < cols; col += blockDim.x) {
  59:       val = dist[row * cols + col];
  60:       CUDA_KERNEL_ASSERT(!(val < zero)); // ! < 0 for NaN handling
  61:       sum = sum + val;
  62:     }
  63: 
  64:     sum = cuda_utils::BlockReduceSum(sum, smem);
  65:     if (threadIdx.x == 0) {
  66:       CUDA_KERNEL_ASSERT(!(val < zero)); // ! < 0 for NaN handling
  67:       smem[0] = sum;
  68:     }
  69:     __syncthreads();
  70: 
  71:     sum = smem[0];
  72:     if (sum > zero) {
  73:       for (int64_t col = threadIdx.x; col < cols; col += blockDim.x) {
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 74-78
```cpp
  74:         dist[row * cols + col] = dist[row * cols + col] / sum;
  75:       }
  76:     }
  77:   }
  78: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 80-101
```cpp
  80: void renormRows(Tensor& t) {
  81:   TORCH_CHECK(t.dim() == 2);
  82:   int64_t rows = t.size(0);
  83:   int64_t cols = t.size(1);
  84: 
  85:   auto props = at::cuda::getCurrentDeviceProperties();
  86:   TORCH_CHECK(props != nullptr);
  87:   int numSM = props->multiProcessorCount;
  88:   const int64_t maxThreads = std::min(
  89:       props->maxThreadsPerBlock, cuda_utils::kCUDABlockReduceMaxThreads());
  90: 
  91:   int warp_size = at::cuda::warp_size();
  92:   dim3 grid(rows < numSM * 4 ? rows : numSM * 4);
  93:   dim3 block(std::min(maxThreads, warp_size * ceil_div(cols, int64_t{warp_size})));
  94: 
  95:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, t.scalar_type(), "renormRows_cuda", [&] {
  96:     renormRowsL1<scalar_t>
  97:         <<<grid, block, (block.x / warp_size) * sizeof(scalar_t),
  98:         at::cuda::getCurrentCUDAStream()>>>(t.mutable_data_ptr<scalar_t>(),
  99:             rows, cols);
 100:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 101:   });
```
- EN: This block defines or continues the implementation of `renormRows`.
- CN: 该代码块定义或继续实现 `renormRows`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 102-102
```cpp
 102: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 104-125
```cpp
 104: template <typename scalar_t>
 105: __device__ int binarySearchForMultinomial(const scalar_t* cumdist,
 106:                                           const scalar_t* dist,
 107:                                           int size,
 108:                                           scalar_t val) {
 109:   int start = 0;
 110:   int end = size;
 111:   // cumdist[size - 1] = 0 => all zero prob dist
 112:   CUDA_KERNEL_ASSERT(cumdist[size - 1] > static_cast<scalar_t>(0));
 113: 
 114:   while (end - start > 0) {
 115:     int mid = start + (end - start) / 2;
 116: 
 117:     scalar_t midVal = cumdist[mid];
 118:     if (midVal < val) {
 119:       start = mid + 1;
 120:     } else {
 121:       end = mid;
 122:     }
 123:   }
 124: 
 125:   if (start == size) {
```
- EN: This block defines or continues the implementation of `binarySearchForMultinomial`.
- CN: 该代码块定义或继续实现 `binarySearchForMultinomial`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 126-130
```cpp
 126:     // No probability mass or precision problems; just return the
 127:     // first non-zero element by setting start to size-1 here,
 128:     // the code below will move it to the last non-zero probability
 129:     // this actually can happen when the random number is 1
 130:     // (github pytorch issue #4858).
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 131-132
```cpp
 131:     start = size - 1;
 132:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 134-134
```cpp
 134:   while(start >= 1 && dist[start] == 0) start--;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 136-137
```cpp
 136:   return start;
 137: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 139-160
```cpp
 139: template <typename scalar_t>
 140: __global__ void
 141: sampleMultinomialWithReplacement(PhiloxCudaState philox_args,
 142:                                  int totalSamples,
 143:                                  int64_t* dest,
 144:                                  int64_t distributions,
 145:                                  int categories,
 146:                                  const scalar_t* normDistPrefixSum,
 147:                                  const scalar_t* normDist) {
 148:   // At the moment, each warp computes one sample value in the binary
 149:   // search due to divergence. It seems possible to compute multiple
 150:   // values and limit divergence though later on.
 151: 
 152:   auto seeds = at::cuda::philox::unpack(philox_args);
 153: 
 154:   // global index formula for 2D grid of 1D blocks
 155:   int idx = blockIdx.y * gridDim.x * blockDim.x + blockIdx.x * blockDim.x + threadIdx.x;
 156: 
 157:   curandStatePhilox4_32_10_t state;
 158:   curand_init(std::get<0>(seeds),
 159:               idx,
 160:               std::get<1>(seeds),
```
- EN: This block defines GPU kernel entry point(s) `sampleMultinomialWithReplacement`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `sampleMultinomialWithReplacement`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 161-161
```cpp
 161:               &state);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 163-163
```cpp
 163:   // The block determines the distribution for which we generate a point
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 164-185
```cpp
 164:   for (int64_t curDist = blockIdx.y;
 165:        curDist < distributions;
 166:        curDist += gridDim.y) {
 167:     for (int sample = blockIdx.x*blockDim.x + threadIdx.x;
 168:          sample < totalSamples; sample += blockDim.x*gridDim.x) {
 169: 
 170:       //we are losing 3 out of 4 generated numbers but it's ok
 171:       //this kernel is not very efficient anyway
 172:       auto rand = curand_uniform4(&state);
 173:       scalar_t r = static_cast<scalar_t>(rand.x);
 174: 
 175:       // Find the bucket that a uniform sample lies in
 176:       int choice = binarySearchForMultinomial<scalar_t>(
 177:           normDistPrefixSum + curDist * categories,
 178:           normDist + curDist * categories,
 179:           categories,
 180:           r);
 181: 
 182:       dest[curDist * totalSamples + sample] = choice;
 183: 
 184:     }
 185:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 186-186
```cpp
 186: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 188-209
```cpp
 188: template <typename scalar_t, typename accscalar_t>
 189: C10_LAUNCH_BOUNDS_1(cuda::detail::CUDA_NUM_THREADS)
 190: __global__ void sampleMultinomialOnce(
 191:     int64_t* dest,
 192:     int64_t distributions,
 193:     int categories,
 194:     const scalar_t* sampled,
 195:     const scalar_t* dist,
 196:     int stride_dist, // dist->stride(0)
 197:     int stride_categories // dist->stride(1)
 198: ) {
 199:   extern __shared__  unsigned char my_smem[];
 200:   __shared__ bool found;
 201:   __shared__ unsigned foundPos;
 202: 
 203:   accscalar_t *smem = reinterpret_cast<accscalar_t *>(my_smem);
 204: 
 205:   accscalar_t accZero = static_cast<accscalar_t>(0);
 206:   scalar_t zero = static_cast<scalar_t>(0);
 207: 
 208:   for (int64_t curDist = blockIdx.x;
 209:        curDist < distributions; curDist += gridDim.x) {
```
- EN: This block defines GPU kernel entry point(s) `sampleMultinomialOnce`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `sampleMultinomialOnce`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 210-211
```cpp
 210:     // Each block handles one distribution
 211:     // First pass, find the total sum of the distribution
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 212-220
```cpp
 212:     accscalar_t sum = accZero;
 213:     scalar_t val;
 214:     for (int cat = threadIdx.x; cat < categories; cat += blockDim.x) {
 215:       val = dist[curDist * stride_dist + cat * stride_categories];
 216:       CUDA_KERNEL_ASSERT(!at::_isnan(val));
 217:       CUDA_KERNEL_ASSERT(!_isinf(val));
 218:       CUDA_KERNEL_ASSERT(!(val < zero));
 219:       sum = sum + static_cast<accscalar_t>(val);
 220:     }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 222-222
```cpp
 222:     // threadIdx.x == 0 has the sum value from this
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 223-223
```cpp
 223:     sum = cuda_utils::BlockReduceSum(sum, smem);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 225-225
```cpp
 225:     // Broadcast sum and sample value
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 226-235
```cpp
 226:     if (threadIdx.x == 0) {
 227:       // Make sure the sum of our distribution didn't overflow
 228:       CUDA_KERNEL_ASSERT(!_isinf(val));
 229:       CUDA_KERNEL_ASSERT(sum > accZero);
 230: 
 231:       foundPos = 0;
 232:       smem[0] = sum;
 233:       smem[1] = sampled[curDist];
 234:     }
 235:     __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 237-239
```cpp
 237:     sum = smem[0];
 238:     scalar_t sample = static_cast<scalar_t>(smem[1]);
 239:     __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 241-248
```cpp
 241:     if (sum == accZero) {
 242:       // Choose the first element
 243:       if (threadIdx.x == 0) {
 244:         dest[curDist] = 0;
 245:       }
 246: 
 247:       continue;
 248:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 250-252
```cpp
 250:     int chunks = (categories + (int)blockDim.x - 1) / blockDim.x;
 251:     accscalar_t prevHighProb = accZero;
 252:     found = false;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 254-275
```cpp
 254:     for (int chunk = 0; chunk < chunks && !found; ++chunk) {
 255:       // All threads in bounds load a value
 256:       int cat = chunk * blockDim.x + threadIdx.x;
 257: 
 258:       accscalar_t dist_val = cat < categories ?
 259:                              static_cast<accscalar_t>(dist[curDist * stride_dist + cat * stride_categories]) / sum :
 260:                              accZero;
 261: 
 262:       smem[threadIdx.x] = dist_val;
 263:       __syncthreads();
 264: 
 265:       // Perform an inclusive prefix sum of the shared memory contents
 266:       for (int offset = 1; offset < blockDim.x; offset *= 2) {
 267:         accscalar_t val = accZero;
 268: 
 269:         if (threadIdx.x >= offset) {
 270:           val = smem[threadIdx.x - offset] + smem[threadIdx.x];
 271:         }
 272: 
 273:         __syncthreads();
 274:         if (threadIdx.x >= offset) {
 275:           smem[threadIdx.x] = val;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 276-278
```cpp
 276:         }
 277:         __syncthreads();
 278:       }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 280-281
```cpp
 280:       // Each thread will check to see if the sample falls in its
 281:       // bucket
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 282-291
```cpp
 282:       scalar_t curBucket =
 283:           static_cast<scalar_t>(smem[threadIdx.x] + prevHighProb);
 284:       scalar_t prevBucket = static_cast<scalar_t>(
 285:           threadIdx.x == 0 ? prevHighProb
 286:                           : smem[threadIdx.x - 1] + prevHighProb);
 287:       bool inBucket =
 288:           (cat < categories) &&
 289:           (!(sample >= curBucket) &&
 290:           (sample >= prevBucket) &&
 291:           (dist_val > zero));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 293-298
```cpp
 293:       if (inBucket) {
 294:         // We're done; we have the sample
 295:         // Torch indices are 1-based
 296:         atomicMax(&foundPos, cat);
 297:         found = true;
 298:       }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 300-300
```cpp
 300:       // Store the previous scan's high value for future use
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 301-301
```cpp
 301:       prevHighProb = prevHighProb + smem[blockDim.x - 1];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 303-304
```cpp
 303:       __syncthreads();
 304:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 306-323
```cpp
 306:     if (threadIdx.x == 0) {
 307:       if (found) {
 308:           dest[curDist] = foundPos;
 309:       } else {
 310:         // This should address a rare bug where we don't select a valid index. This likely occurs when
 311:         // due to floating point arithmetic rounding errors, our cumulative sum does not add up to 1, but
 312:         // and our uniform sample is greater than this value. In this case we likely have uninitialized memory
 313:         // in dest[curDist]. So basically we will loop through the distribution and pick the largest index
 314:         // where the distribution is non-zero. This is obviously terribly inefficient, but due to the
 315:         // rarity in which this occurs, this should not be an issue.
 316:         for (int cat = categories - 1; cat >= 0; --cat) {
 317:           if (dist[curDist * stride_dist + cat * stride_categories] > zero) {
 318:             dest[curDist] = cat;
 319:             break;
 320:           }
 321:         }
 322:       }
 323:     }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 324-325
```cpp
 324:   }
 325: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 327-348
```cpp
 327: void multinomial_with_replacement_kernel_impl(
 328:     Tensor& result,
 329:     const Tensor& self,
 330:     const int64_t n_sample,
 331:     std::optional<Generator> generator) {
 332:   auto gen = get_generator_or_default<CUDAGeneratorImpl>(generator, cuda::detail::getDefaultCUDAGenerator());
 333: 
 334:   int inputSize = self.dim();
 335:   int64_t numDist =
 336:       inputSize == 1 ? 1 : self.size(0);
 337:   int numCategories =
 338:       inputSize == 1 ? self.size(0) : self.size(1);
 339: 
 340:   // Restructure data for 2d
 341:   auto self_v = inputSize == 1 ? self.view({numDist, numCategories}) : self;
 342: 
 343:   result.resize_({numDist, n_sample});
 344: 
 345:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, self_v.scalar_type(), "multinomial_kernel_cuda", [&] {
 346:     using accscalar_t = at::acc_type<scalar_t, true>;
 347:     auto props = at::cuda::getCurrentDeviceProperties();
 348:     TORCH_CHECK(props != nullptr);
```
- EN: This block defines or continues the implementation of `multinomial_with_replacement_kernel_impl`.
- CN: 该代码块定义或继续实现 `multinomial_with_replacement_kernel_impl`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 349-351
```cpp
 349:     int numSM = props->multiProcessorCount;
 350:     int maxThreads = props->maxThreadsPerBlock;
 351:     int maxShared = props->sharedMemPerBlock;
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 353-356
```cpp
 353:     int warp_size = at::cuda::warp_size();
 354:     int requiredWarps = at::ceil_div(numCategories, warp_size);
 355:     int requiredThreads = std::min(maxThreads, requiredWarps * warp_size);
 356:     int requiredShared = requiredThreads * sizeof(accscalar_t);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 358-379
```cpp
 358:     if (n_sample == 1 && maxShared >= requiredShared) {
 359:       // Optimized allocation-free implementation
 360:       // To exploit greater parallelism for the sampling, generate the
 361:       // Uniform random samples in a separate kernel launch, into
 362:       // temporarily allocated memory. The device RNG is thread-limited
 363:       Tensor sampled = at::detail::empty_cuda({numDist, n_sample}, self_v.options());
 364:       at::native::uniform_(sampled, 0.0, 1.0, generator);
 365: 
 366:       dim3 block(requiredThreads);
 367:       dim3 grid(std::min(static_cast<int>(numDist), numSM * 4));
 368: 
 369:       sampleMultinomialOnce<scalar_t, accscalar_t>
 370:           <<<grid, block,
 371:           requiredShared,
 372:           at::cuda::getCurrentCUDAStream()>>>(
 373:               result.mutable_data_ptr<int64_t>(),
 374:                   numDist,
 375:                   numCategories,
 376:                   sampled.const_data_ptr<scalar_t>(),
 377:                   self_v.const_data_ptr<scalar_t>(),
 378:                   self_v.stride(0),
 379:                   self_v.stride(1)
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 380-383
```cpp
 380:           );
 381:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 382:     } else {
 383:       // Generic, slow implementation with memory allocations
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 385-386
```cpp
 385:       // For sampling without replacement, we modify the distribution
 386:       // for subsequent samples in this space
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 387-394
```cpp
 387:       Tensor origDist = native::empty_like(
 388:           self_v,
 389:           std::nullopt /* dtype */,
 390:           std::nullopt /* layout */,
 391:           std::nullopt /* device */,
 392:           std::nullopt /* pin_memory */,
 393:           LEGACY_CONTIGUOUS_MEMORY_FORMAT);
 394:       origDist.copy_(self_v);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 396-402
```cpp
 396:       Tensor normDist = native::empty_like(
 397:           self_v,
 398:           std::nullopt /* dtype */,
 399:           std::nullopt /* layout */,
 400:           std::nullopt /* device */,
 401:           std::nullopt /* pin_memory */,
 402:           LEGACY_CONTIGUOUS_MEMORY_FORMAT);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 404-410
```cpp
 404:       Tensor prefixSum = native::empty_like(
 405:           self_v,
 406:           std::nullopt /* dtype */,
 407:           std::nullopt /* layout */,
 408:           std::nullopt /* device */,
 409:           std::nullopt /* pin_memory */,
 410:           LEGACY_CONTIGUOUS_MEMORY_FORMAT);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 412-412
```cpp
 412:       // Renorm along rows
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 413-414
```cpp
 413:       normDist.copy_(origDist);
 414:       renormRows(normDist);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 416-416
```cpp
 416:       // Prefix sum along rows
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 417-417
```cpp
 417:       at::cuda::cumsum_out(prefixSum, normDist, 1);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 419-419
```cpp
 419:       PhiloxCudaState rng_engine_inputs;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 421-423
```cpp
 421:         // Binary search is warp divergent (so effectively we're running
 422:         // with just a single thread), but for better utilization,
 423:         // we need each block to have at least 4 warps.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 424-424
```cpp
 424:         dim3 block(128);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 426-427
```cpp
 426:         // Each block will generate a sample from one
 427:         // distribution concurrently.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 428-440
```cpp
 428:         int grid_y=std::min<int>(numDist, at::cuda::getCurrentDeviceProperties()->maxGridSize[1]);
 429:         dim3 grid((n_sample-1)/block.x+1, grid_y);
 430:         {
 431:           // See Note [Acquire lock when using random generators]
 432:           std::lock_guard<std::mutex> lock(gen->mutex_);
 433: 
 434:           // each thread generates a single sample for (numdist/numblocks.y) distributions, however, since we have to use
 435:           // curand_uniform4 (See Note [Register spilling in curand call for CUDA < 10]),
 436:           // offset is 4 times that.
 437:           auto offset = ((numDist-1)/grid.y+1)*4;
 438:           rng_engine_inputs = gen->philox_cuda_state(offset);
 439:         }
 440:         // Sample with replacement
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 442-452
```cpp
 442:         sampleMultinomialWithReplacement
 443:             <<<grid, block, 0, at::cuda::getCurrentCUDAStream()>>>(
 444:                 rng_engine_inputs,
 445:                 n_sample,
 446:                 result.mutable_data_ptr<int64_t>(),
 447:                 numDist, numCategories,
 448:                 prefixSum.const_data_ptr<scalar_t>(),
 449:                 normDist.const_data_ptr<scalar_t>());
 450:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 451:     }
 452:   });
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 454-458
```cpp
 454:   if (inputSize == 1) {
 455:     result.resize_({n_sample});
 456:   }
 457: }
 458: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 460-463
```cpp
 460: REGISTER_DISPATCH(
 461:     multinomial_with_replacement_stub,
 462:     &multinomial_with_replacement_kernel_impl);
 463: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: The registration macro binds this implementation into PyTorch dispatch so higher-level operators can call the CUDA specialization.
- CN: 注册宏把该实现接入 PyTorch 分发系统，使上层算子能够调用这个 CUDA 特化版本。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- Dispatch registration exposes the CUDA specialization to higher PyTorch layers. / 分发注册把 CUDA 特化实现暴露给更高层的 PyTorch 调用链。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/AccumulateType.h>`
  - `<ATen/ceil_div.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/Utils.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/EmptyTensor.h>`
  - `<ATen/cuda/detail/KernelUtils.h>`
  - `<ATen/native/UnaryOps.h>`
  - `<ATen/native/cuda/LaunchUtils.h>`
  - `<ATen/cuda/CUDAGraphsUtils.cuh>`
  - `<ATen/native/cuda/block_reduce.cuh>`
- Runtime symbols / 运行时符号:
  - `multinomial_with_replacement_stub`
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `REGISTER_DISPATCH`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::warp_size`
  - `at::cuda::getCurrentCUDAStream`
  - `at::cuda::philox::unpack`
  - `at::cuda::cumsum_out`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
