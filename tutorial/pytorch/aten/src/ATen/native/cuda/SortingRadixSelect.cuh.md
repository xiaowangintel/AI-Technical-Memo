# SortingRadixSelect.cuh — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/SortingRadixSelect.cuh`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Declares or defines CUDA helpers/templates associated with `convert`, `deconvert`, `constexpr`, `findPattern`.
- 用途（中文）: 声明或定义与 `convert`, `deconvert`, `constexpr`, `findPattern` 相关的 CUDA 辅助函数/模板。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6
```cpp
   1: #include <ATen/ceil_div.h>
   2: #include <c10/macros/Macros.h>
   3: #include <ATen/cuda/AsmUtils.cuh>
   4: #include <ATen/cuda/Atomic.cuh>
   5: #include <ATen/cuda/DeviceUtils.cuh>
   6: #include <type_traits>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/ceil_div.h>`, `<c10/macros/Macros.h>`, `<ATen/cuda/AsmUtils.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/ceil_div.h>`, `<c10/macros/Macros.h>`, `<ATen/cuda/AsmUtils.cuh>`。

### Lines 8-29
```cpp
   8: namespace at::native {
   9: 
  10: template <typename scalar_t>
  11: struct TopKTypeConfig {};
  12: 
  13: template <>
  14: struct TopKTypeConfig<float> {
  15:   typedef uint32_t RadixType;
  16: 
  17:   // Converts a float to an integer representation with the same
  18:   // sorting; i.e., for floats f1, f2:
  19:   // if f1 < f2 then convert(f1) < convert(f2)
  20:   // We use this to enable radix selection of floating-point values.
  21:   // This also gives a relative order for NaNs, but that's ok, as they
  22:   // will all be adjacent
  23:   // neg inf: signbit=1 exp=ff fraction=0 --> radix = 0 00 ff..
  24:   // pos inf: signbit=0 exp=ff fraction=0 --> radix = 1 ff 00..
  25:   // pos nan: signbit=0 exp=ff fraction>0 --> radix = 1 ff x>0
  26:   // neg nan: signbit=1 exp=ff fraction>0 --> radix = 0 00 x<ff...
  27:   static inline __device__ RadixType convert(float v) {
  28:     RadixType x = __float_as_int(v);
  29:     RadixType mask = (x & 0x80000000) ? 0xffffffff : 0x80000000;
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `convert`.
- CN: 该代码块定义或继续实现 `convert`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 31-32
```cpp
  31:     return (v == v) ? (x ^ mask) : 0xffffffff;
  32:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 34-39
```cpp
  34:   static inline __device__ float deconvert(RadixType v) {
  35:     RadixType mask = (v & 0x80000000) ? 0x80000000 : 0xffffffff;
  36: 
  37:     return __int_as_float(v ^ mask);
  38:   }
  39: };
```
- EN: This block defines or continues the implementation of `deconvert`.
- CN: 该代码块定义或继续实现 `deconvert`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 41-52
```cpp
  41: template <>
  42: struct TopKTypeConfig<uint8_t> {
  43:   typedef uint32_t RadixType;
  44: 
  45:   static inline __device__ RadixType convert(uint8_t v) {
  46:     return v;
  47:   }
  48: 
  49:   static inline __device__ uint8_t deconvert(RadixType v) {
  50:     return v;
  51:   }
  52: };
```
- EN: This block defines or continues the implementation of `convert`, `deconvert`.
- CN: 该代码块定义或继续实现 `convert`, `deconvert`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 54-65
```cpp
  54: template <>
  55: struct TopKTypeConfig<int8_t> {
  56:   typedef uint32_t RadixType;
  57: 
  58:   static inline __device__ RadixType convert(int8_t v) {
  59:     return 128u + v;
  60:   }
  61: 
  62:   static inline __device__ int8_t deconvert(RadixType v) {
  63:     return v - 128;
  64:   }
  65: };
```
- EN: This block defines or continues the implementation of `convert`, `deconvert`.
- CN: 该代码块定义或继续实现 `convert`, `deconvert`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 67-79
```cpp
  67: template <>
  68: struct TopKTypeConfig<int16_t> {
  69:   typedef uint32_t RadixType;
  70: 
  71:   static inline __device__ RadixType convert(int16_t v) {
  72:     static_assert(sizeof(short) == 2, "");
  73:     return 32768u + v;
  74:   }
  75: 
  76:   static inline __device__ int16_t deconvert(RadixType v) {
  77:     return v - 32768;
  78:   }
  79: };
```
- EN: This block defines or continues the implementation of `convert`, `deconvert`.
- CN: 该代码块定义或继续实现 `convert`, `deconvert`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 81-93
```cpp
  81: template <>
  82: struct TopKTypeConfig<int32_t> {
  83:   typedef uint32_t RadixType;
  84: 
  85:   static inline __device__ RadixType convert(int32_t v) {
  86:     static_assert(sizeof(int) == 4, "");
  87:     return 2147483648u + v;
  88:   }
  89: 
  90:   static inline __device__ int32_t deconvert(RadixType v) {
  91:     return v - 2147483648u;
  92:   }
  93: };
```
- EN: This block defines or continues the implementation of `convert`, `deconvert`.
- CN: 该代码块定义或继续实现 `convert`, `deconvert`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 95-107
```cpp
  95: template <>
  96: struct TopKTypeConfig<int64_t> {
  97:   typedef uint64_t RadixType;
  98: 
  99:   static inline __device__ RadixType convert(int64_t v) {
 100:     static_assert(sizeof(int64_t) == 8, "");
 101:     return 9223372036854775808ull + v;
 102:   }
 103: 
 104:   static inline __device__ int64_t deconvert(RadixType v) {
 105:     return v - 9223372036854775808ull;
 106:   }
 107: };
```
- EN: This block defines or continues the implementation of `convert`, `deconvert`.
- CN: 该代码块定义或继续实现 `convert`, `deconvert`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 109-123
```cpp
 109: template <>
 110: struct TopKTypeConfig<double> {
 111:   typedef uint64_t RadixType;
 112: 
 113:   static inline __device__ RadixType convert(double v) {
 114:     RadixType x = __double_as_longlong(v);
 115:     RadixType mask = -((x >> 63)) | 0x8000000000000000;
 116:     return (v == v) ? (x ^ mask) : 0xffffffffffffffff;
 117:   }
 118: 
 119:   static inline __device__ double deconvert(RadixType v) {
 120:     RadixType mask = ((v >> 63) - 1) | 0x8000000000000000;
 121:     return __longlong_as_double(v ^ mask);
 122:   }
 123: };
```
- EN: This block defines or continues the implementation of `convert`, `deconvert`.
- CN: 该代码块定义或继续实现 `convert`, `deconvert`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 125-139
```cpp
 125: template <>
 126: struct TopKTypeConfig<at::Half> {
 127:   typedef uint32_t RadixType;
 128: 
 129:   static inline __device__ RadixType convert(at::Half v) {
 130:     RadixType x = __half_as_ushort(v);
 131:     RadixType mask = (x & 0x00008000) ? 0x0000ffff : 0x00008000;
 132:     return (v == v) ? (x ^ mask) : 0xffff;
 133:   }
 134: 
 135:   static inline __device__ at::Half deconvert(RadixType v) {
 136:     RadixType mask = (v & 0x00008000) ? 0x00008000 : 0x0000ffff;
 137:     return __ushort_as_half(v ^ mask);
 138:   }
 139: };
```
- EN: This block defines or continues the implementation of `convert`, `deconvert`.
- CN: 该代码块定义或继续实现 `convert`, `deconvert`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 141-157
```cpp
 141: template <>
 142: struct TopKTypeConfig<at::BFloat16> {
 143:   typedef uint32_t RadixType;
 144: 
 145:   static inline __device__ RadixType convert(at::BFloat16 v) {
 146:     RadixType x = v.x;
 147:     RadixType mask = (x & 0x00008000) ? 0x0000ffff : 0x00008000;
 148:     return (v == v) ? (x ^ mask) : 0xffff;
 149:   }
 150: 
 151:   static inline __device__ at::BFloat16 deconvert(RadixType v) {
 152:     RadixType mask = (v & 0x00008000) ? 0x00008000 : 0x0000ffff;
 153:     at::BFloat16 r;
 154:     r.x = (v ^ mask);
 155:     return r;
 156:   }
 157: };
```
- EN: This block defines or continues the implementation of `convert`, `deconvert`.
- CN: 该代码块定义或继续实现 `convert`, `deconvert`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 159-159
```cpp
 159: // Over what radix we are selecting values
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 160-162
```cpp
 160: constexpr int RADIX_BITS = 2; // digits are base-(2 ^ RADIX_BITS)
 161: constexpr int RADIX_SIZE = 4; // 2 ^ RADIX_BITS
 162: constexpr int RADIX_MASK = (RADIX_SIZE - 1);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 164-164
```cpp
 164: #ifndef USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 165-169
```cpp
 165: // This function counts the distribution of all input values in a
 166: // slice we are selecting by radix digit at `radixDigitPos`, but only
 167: // those that pass the filter `((v & desiredMask) == desired)`.
 168: // This produces and broadcasts the seen counts for a single block only.
 169: // `smem` must have at least `RadixSize` elements.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 170-183
```cpp
 170: template <
 171:     typename scalar_t,
 172:     typename bitwise_t,
 173:     typename index_t,
 174:     typename CountType,
 175:     int RadixSize,
 176:     int RadixBits>
 177: __device__ void countRadixUsingMask(
 178:     CountType counts[RadixSize],
 179:     CountType* smem,
 180:     bitwise_t desired,
 181:     bitwise_t desiredMask,
 182:     int radixDigitPos,
 183:     index_t sliceSize,
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 184-205
```cpp
 184:     index_t withinSliceStride,
 185:     const scalar_t* data) {
 186:   // Clear out per-thread counts from a previous round
 187: #pragma unroll
 188:   for (int i = 0; i < RadixSize; ++i) {
 189:     counts[i] = 0;
 190:   }
 191: 
 192:   if (threadIdx.x < RadixSize) {
 193:     smem[threadIdx.x] = 0;
 194:   }
 195:   __syncthreads();
 196: 
 197:   // Scan over all the data. Upon a read, the warp will accumulate
 198:   // counts per each digit in the radix using warp voting.
 199:   // Must be called outside of loop to ensure all threads participate
 200:   unsigned mask = WARP_BALLOT(threadIdx.x < sliceSize);
 201:   for (index_t i = threadIdx.x; i < sliceSize;) {
 202:     bitwise_t val =
 203:         TopKTypeConfig<scalar_t>::convert(doLdg(&data[i * withinSliceStride]));
 204: 
 205:     bool hasVal = ((val & desiredMask) == desired);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 206-207
```cpp
 206:     bitwise_t digitInRadix = at::cuda::Bitfield<bitwise_t>::getBitfield(
 207:         val, radixDigitPos, RadixBits);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 209-209
```cpp
 209: #pragma unroll
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 210-216
```cpp
 210:     for (uint32_t j = 0; j < RadixSize; ++j) {
 211:       bool vote = hasVal && (digitInRadix == j);
 212:       counts[j] += __popc(WARP_BALLOT(vote, mask));
 213:     }
 214:     i += blockDim.x;
 215:     mask = WARP_BALLOT(i < sliceSize, mask);
 216:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 218-220
```cpp
 218:   // Now, for each warp, sum values
 219:   // Note: uint64_t on Linux is unsigned long, but CUDA atomicAdd expects
 220:   // unsigned long long. We use reinterpret_cast for compatibility.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 221-231
```cpp
 221:   if (at::cuda::getLaneId() == 0) {
 222: #pragma unroll
 223:     for (uint32_t i = 0; i < RadixSize; ++i) {
 224:       if constexpr (std::is_same_v<CountType, uint64_t>) {
 225:         atomicAdd(reinterpret_cast<unsigned long long*>(smem) + i,
 226:                   static_cast<unsigned long long>(counts[i]));
 227:       } else {
 228:         atomicAdd(&smem[i], counts[i]);
 229:       }
 230:     }
 231:   }
```
- EN: This block defines or continues the implementation of `constexpr`.
- CN: 该代码块定义或继续实现 `constexpr`。

### Lines 233-233
```cpp
 233:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 235-235
```cpp
 235:   // For each thread, read in the total counts
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 236-236
```cpp
 236: #pragma unroll
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 237-239
```cpp
 237:   for (uint32_t i = 0; i < RadixSize; ++i) {
 238:     counts[i] = smem[i];
 239:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 241-242
```cpp
 241:   __syncthreads();
 242: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 244-245
```cpp
 244: // This finds the unique value `v` that matches the pattern
 245: // ((v & desired) == desiredMask) in our sorted int format
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 246-267
```cpp
 246: template <typename scalar_t, typename bitwise_t, typename index_t>
 247: __device__ scalar_t findPattern(
 248:     scalar_t* smem,
 249:     const scalar_t* data,
 250:     index_t sliceSize,
 251:     index_t withinSliceStride,
 252:     bitwise_t desired,
 253:     bitwise_t desiredMask) {
 254:   if (threadIdx.x < 2) {
 255:     smem[threadIdx.x] = static_cast<scalar_t>(0);
 256:   }
 257:   __syncthreads();
 258: 
 259:   // All threads participate in the loop, in order to sync on the flag
 260:   index_t numIterations = round_up(sliceSize, static_cast<index_t>(blockDim.x));
 261:   for (index_t i = threadIdx.x; i < numIterations; i += blockDim.x) {
 262:     bool inRange = (i < sliceSize);
 263:     scalar_t v = inRange ? doLdg(&data[i * withinSliceStride])
 264:                          : static_cast<scalar_t>(0);
 265: 
 266:     if (inRange &&
 267:         ((TopKTypeConfig<scalar_t>::convert(v) & desiredMask) == desired)) {
```
- EN: This block defines or continues the implementation of `findPattern`.
- CN: 该代码块定义或继续实现 `findPattern`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 268-269
```cpp
 268:       // There should not be conflicts if we are using findPattern,
 269:       // since the result is unique
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 270-272
```cpp
 270:       smem[0] = static_cast<scalar_t>(1);
 271:       smem[1] = v; // can't use val as the flag, since it could be 0
 272:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 274-274
```cpp
 274:     __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 276-277
```cpp
 276:     scalar_t found = smem[0];
 277:     scalar_t val = smem[1];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 279-279
```cpp
 279:     __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 281-281
```cpp
 281:     // Check to see if a thread found the value
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 282-286
```cpp
 282:     if (found != static_cast<scalar_t>(0)) {
 283:       // all threads return this value
 284:       return val;
 285:     }
 286:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 288-288
```cpp
 288:   // should not get here
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 289-291
```cpp
 289:   CUDA_KERNEL_ASSERT(false);
 290:   return static_cast<scalar_t>(0);
 291: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 293-293
```cpp
 293: #else
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 295-295
```cpp
 295: /*
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 296-309
```cpp
 296: This implementation of radixSelect optimizes the k-th element selection
 297: algorithm by dynamically utilizing shared memory to cache input data when
 298: possible, significantly reducing global memory traffic during the iterative bit
 299: discovery process. The radixSelect algorithm finds the k-th element by
 300: iteratively uncovering its bit pattern through multiple passes over the data.
 301: Each pass determines 2 bits of the target value's bitmap (up to 16 passes for
 302: float32 inputs). As iterations progress, the number of relevant values decreases
 303: by approximately 4× per pass, assuming uniform bit distribution. While initially
 304: the input data may be too large to fit in shared memory, it often becomes
 305: cacheable after a few filtering iterations as the data size shrinks. This
 306: implementation introduces dynamic shared memory caching that checks at each
 307: iteration whether the filtered data fits within available LDS (a few KB's
 308: allocated for this purpose). When the data fits, it is cached to shared memory,
 309: eliminating redundant global memory reads in subsequent operations within that
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 310-315
```cpp
 310: iteration. New kernel functions countRadixUsingMaskDataSmem and findPatternSmem
 311: were introduced to seamlessly handle both cached (LDS) and non-cached (global
 312: memory) data paths. These variants maintain backward compatibility with the
 313: original algorithm and automatically fall back to global memory access when data
 314: exceeds LDS capacity.
 315: */
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 317-321
```cpp
 317: // this is the main loop of the countRadixUsingMask function that counts the
 318: // distribution of the bits in the radix digit at `radixDigitPos` to
 319: // `radixDigitPos`+RADIX_BITS-1. DataAccessor is a function that returns the
 320: // input data value at index i. It could potentially be a global memory accessor
 321: // or a shared memory accessor.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 322-335
```cpp
 322: template <
 323:     typename scalar_t,
 324:     typename bitwise_t,
 325:     typename index_t,
 326:     typename CountType,
 327:     int RadixSize,
 328:     int RadixBits,
 329:     bool prefetch,
 330:     typename DataAccessor>
 331: __device__ __forceinline__ void countRadixLoop(
 332:     CountType counts[RadixSize], // counts[i] will be the number of matching
 333:                                  // elements ((val & desiredMask) == desired)
 334:                                  // that have the digits [radixDigitPos,
 335:                                  // radixDigitPos+RADIX_BITS-1] set to i.
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 336-356
```cpp
 336:     bitwise_t
 337:         desired, // combined with desiredMask to filter relevant elements. A
 338:                  // value is relevant if ((val & desiredMask) == desired).
 339:     bitwise_t
 340:         desiredMask, // combined with desired to filter relevant elements. A
 341:                      // value is relevant if ((val & desiredMask) == desired).
 342:     int radixDigitPos, // the position of the radix digit.
 343:     index_t loopBound, // the upper bound of the loop.
 344:     DataAccessor&& getData) { // a function that returns the input data value at
 345:                               // index i. It could potentially be a global
 346:                               // memory accessor or a shared memory accessor.
 347: 
 348:   // the kernel consists of two parts:
 349:   // phase 1: processing 4 elements at an iteration.
 350:   // phase 2: processing 1 element at an iteration.
 351: 
 352:   constexpr index_t unroll_factor = 4;
 353:   index_t unroll_segment =
 354:       (loopBound / (blockDim.x * unroll_factor)) * blockDim.x * unroll_factor;
 355: 
 356:   // phase 1: processing 4 elements at an iteration.
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 358-379
```cpp
 358:   for (index_t i = threadIdx.x * unroll_factor; i < unroll_segment;
 359:        i += blockDim.x * unroll_factor) {
 360: 
 361:     // prefetch 4 elements.
 362:     scalar_t v0 = getData(i);
 363:     scalar_t v1 = getData(i + 1);
 364:     scalar_t v2 = getData(i + 2);
 365:     scalar_t v3 = getData(i + 3);
 366: 
 367:     // convert the values to bitwise_t.
 368:     bitwise_t val0 = TopKTypeConfig<scalar_t>::convert(v0);
 369:     bitwise_t val1 = TopKTypeConfig<scalar_t>::convert(v1);
 370:     bitwise_t val2 = TopKTypeConfig<scalar_t>::convert(v2);
 371:     bitwise_t val3 = TopKTypeConfig<scalar_t>::convert(v3);
 372: 
 373:     // check if the values match the desired pattern.
 374:     bool hasVal0 = ((val0 & desiredMask) == desired);
 375:     bool hasVal1 = ((val1 & desiredMask) == desired);
 376:     bool hasVal2 = ((val2 & desiredMask) == desired);
 377:     bool hasVal3 = ((val3 & desiredMask) == desired);
 378: 
 379:     // get the bits [radixDigitPos, radixDigitPos+RADIX_BITS-1] of the values.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 380-387
```cpp
 380:     bitwise_t digitInRadix0 = at::cuda::Bitfield<bitwise_t>::getBitfield(
 381:         val0, radixDigitPos, RadixBits);
 382:     bitwise_t digitInRadix1 = at::cuda::Bitfield<bitwise_t>::getBitfield(
 383:         val1, radixDigitPos, RadixBits);
 384:     bitwise_t digitInRadix2 = at::cuda::Bitfield<bitwise_t>::getBitfield(
 385:         val2, radixDigitPos, RadixBits);
 386:     bitwise_t digitInRadix3 = at::cuda::Bitfield<bitwise_t>::getBitfield(
 387:         val3, radixDigitPos, RadixBits);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 389-389
```cpp
 389: // counting across the warp.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 390-390
```cpp
 390: #pragma unroll
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 391-403
```cpp
 391:     for (uint32_t j = 0; j < RadixSize; ++j) {
 392:       // checking pattern match & digit match.
 393:       bool vote0 = hasVal0 && (digitInRadix0 == j);
 394:       bool vote1 = hasVal1 && (digitInRadix1 == j);
 395:       bool vote2 = hasVal2 && (digitInRadix2 == j);
 396:       bool vote3 = hasVal3 && (digitInRadix3 == j);
 397: 
 398:       // how many threads in this warp found digitInRadix == j while matching
 399:       // the desired pattern?
 400:       counts[j] += __popcll(WARP_BALLOT(vote0)) + __popcll(WARP_BALLOT(vote1)) +
 401:           __popcll(WARP_BALLOT(vote2)) + __popcll(WARP_BALLOT(vote3));
 402:     }
 403:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 405-408
```cpp
 405:   // phase 2: processing 1 element at an iteration.
 406: 
 407:   // prefetching pattern if prefetch is true.
 408:   // prefetching pattern is only useful for global memory access.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 409-429
```cpp
 409:   scalar_t v_curr;
 410:   if constexpr (prefetch) {
 411:     v_curr = unroll_segment + threadIdx.x < loopBound
 412:         ? getData(unroll_segment + threadIdx.x)
 413:         : static_cast<scalar_t>(0);
 414:   }
 415:   for (index_t i = unroll_segment + threadIdx.x;
 416:        i < loopBound;
 417:        i += blockDim.x) {
 418:         scalar_t v_local; // the current element.
 419:         scalar_t v_next; // the next element. Used for prefetching.
 420: 
 421:         if constexpr (prefetch) {
 422:           // prefetch the next element.
 423:           v_local = v_curr;
 424:           v_next = i + blockDim.x < loopBound ? getData(i + blockDim.x)
 425:                                               : static_cast<scalar_t>(0);
 426:         }
 427:         else {
 428:           v_local = getData(i); // if no prefetching, just get the current element.
 429:         }
```
- EN: This block defines or continues the implementation of `constexpr`.
- CN: 该代码块定义或继续实现 `constexpr`。

### Lines 431-438
```cpp
 431:         bitwise_t val = TopKTypeConfig<scalar_t>::convert(v_local);
 432:         // check if bit pattern matches the pattern we have already discovered for
 433:         // topk value v.
 434:         bool hasVal = ((val & desiredMask) == desired);
 435:         // get the bits [radixDigitPos, radixDigitPos+RADIX_BITS-1] of the value
 436:         // v.
 437:         bitwise_t digitInRadix = at::cuda::Bitfield<bitwise_t>::getBitfield(
 438:             val, radixDigitPos, RadixBits);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 440-440
```cpp
 440: // counting across the warp.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 441-441
```cpp
 441: #pragma unroll
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 442-448
```cpp
 442:     for (uint32_t j = 0; j < RadixSize; ++j) {
 443:       // checking pattern match & digit match.
 444:       bool vote = hasVal && (digitInRadix == j);
 445:       // how many threads in this warp found digitInRadix == j while matching
 446:       // the desired pattern?
 447:       counts[j] += __popcll(WARP_BALLOT(vote));
 448:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 450-454
```cpp
 450:     if constexpr (prefetch) {
 451:       v_curr = v_next; // closing the prefetching loop.
 452:     }
 453:   }
 454: }
```
- EN: This block defines or continues the implementation of `constexpr`.
- CN: 该代码块定义或继续实现 `constexpr`。

### Lines 456-461
```cpp
 456: // Aggregates radix matches across all warps and distributes results back to all threads.
 457: // Uses double-buffering via buffer_index (0 or 1) to alternate between two smem segments,
 458: // preventing race conditions between concurrent iterations. Since countRadixUsingMaskDataSmem
 459: // performs __syncthreads() internally, at most two loop iterations can be in flight
 460: // simultaneously, so two buffers are sufficient. buffer_index is toggled after each
 461: // countRadixUsingMaskDataSmem invocation.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 462-483
```cpp
 462: template <
 463:     typename CountType,
 464:     int RadixSize,
 465:     int RadixBits>
 466: __device__ __forceinline__ void countRadixAggregateCounts(
 467:     CountType counts[RadixSize], // counts[i] will be the number of matching
 468:                                  // elements ((val & desiredMask) == desired)
 469:                                  // that have the digits [radixDigitPos,
 470:                                  // radixDigitPos+RADIX_BITS-1] set to i.
 471:     CountType* smem, // shared memory for inter-warp reduction of counts.
 472:     int buffer_index){ // buffer index for smem.
 473: 
 474:   // Maximum number of warps per workgroup. HIP workgroups have at most 1024 threads.
 475:   // Warp size is at least 32 (can be 64 on some architectures), so we use 32 for safety.
 476:   // This sizes shared memory buffers to accommodate all possible warps: 1024/32 = 32.
 477:   constexpr uint MAX_WARPS = 1024/C10_WARP_SIZE_LOWER_BOUND;
 478:   const int buffer_offset = buffer_index * MAX_WARPS * RadixSize; // offset of the buffer in smem.
 479:   const uint WARP_BITS = __builtin_ctz(C10_WARP_SIZE);
 480: 
 481:   const uint num_warps = blockDim.x >> WARP_BITS;  // Actual number of warps in this block
 482:   const uint warp_id = threadIdx.x >> WARP_BITS; // = threadIdx.x / C10_WARP_SIZE
 483:   const int lane_id = at::cuda::getLaneId(); // = threadIdx.x % C10_WARP_SIZE
```
- EN: This block defines or continues the implementation of `countRadixAggregateCounts`.
- CN: 该代码块定义或继续实现 `countRadixAggregateCounts`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 485-487
```cpp
 485:   // Stage 1: Each warp's lane 0 stores its counts in smem.
 486:   // Layout after Stage 1: [warp0: all radix bins], [warp1: all radix bins], ...
 487:   // this layout starts from index buffer_offset.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 488-497
```cpp
 488:   if (lane_id == 0) {
 489: #pragma unroll
 490:     for (uint32_t i = 0; i < RadixSize; ++i) {
 491:       smem[
 492:             buffer_offset
 493:           + warp_id * RadixSize
 494:           + i
 495:           ] = counts[i];
 496:     }
 497:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 499-499
```cpp
 499:   __syncthreads(); // wait for all warps to finish storing their counts to smem.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 501-503
```cpp
 501:   // Stage 2: Warp0 performs reduction for all bins.
 502:   // Layout after Stage 2: [final radix0 sum], [final radix1 sum], ..., [final radix(RadixSize-1) sum]
 503:   // this layout starts from index buffer_offset.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 504-515
```cpp
 504:   if (warp_id == 0 && lane_id < RadixSize) {
 505:     CountType sum = 0;
 506: #pragma unroll
 507:     for (int w = 0; w < num_warps; ++w) {
 508:       sum += smem[
 509:                     buffer_offset
 510:                   + w * RadixSize
 511:                   + lane_id
 512:                   ];
 513:     }
 514:     smem[buffer_offset + lane_id] = sum;
 515:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 517-517
```cpp
 517:   __syncthreads(); // Wait for warp 0 to finish reduction.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 519-519
```cpp
 519:   // Stage 3: Each thread reads the final counts from smem.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 520-520
```cpp
 520: #pragma unroll
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 521-524
```cpp
 521:   for (uint32_t i = 0; i < RadixSize; ++i) {
 522:     counts[i] = smem[buffer_offset + i];
 523:   }
 524: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 526-532
```cpp
 526: // This function counts the distribution of all input values in a
 527: // slice we are selecting by radix digit at `radixDigitPos`, but only
 528: // those that pass the filter `((v & desiredMask) == desired)`.
 529: // This produces and broadcasts the seen counts for a single block only.
 530: // `smem` must have at least `RadixSize` elements.
 531: // this is an smem-friendly version of the countRadixUsingMask function.
 532: // it works when data is in global memory or in shared memory.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 533-546
```cpp
 533: template <
 534:     typename scalar_t,
 535:     typename bitwise_t,
 536:     typename index_t,
 537:     typename CountType,
 538:     int RadixSize,
 539:     int RadixBits>
 540: __device__ void countRadixUsingMaskDataSmem(
 541:     CountType
 542:         counts[RadixSize], // counts[i] will be the number of matching elements
 543:                            // ((val & desiredMask) == desired) that have the
 544:                            // digits [radixDigitPos, radixDigitPos+RADIX_BITS-1]
 545:                            // set to i in the warp.
 546:     CountType* smem, // shared memory for inter-warp reduction of counts.
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 547-560
```cpp
 547:     int buffer_index, // buffer index for smem.
 548:     bitwise_t
 549:         desired, // combined with desiredMask to filter relevant elements. An
 550:                  // element is relevant if ((val & desiredMask) == desired).
 551:     bitwise_t
 552:         desiredMask, // combined with desired to filter relevant elements. An
 553:                      // element is relevant if ((val & desiredMask) == desired).
 554:     int radixDigitPos, // position of the radix digit.
 555:     index_t sliceSize, // size of the input slice.
 556:     index_t withinSliceStride, // stride of the input slice.
 557:     const scalar_t* data, // input data. This is global memory.
 558:     const scalar_t*
 559:         dataSmem, // input data stored in shared memory. This is shared memory.
 560:                   // It is not initialized if dataSmemSize == 0.
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 561-582
```cpp
 561:     int dataSmemSize) { // input data size stored in shared memory. dataSmemSize
 562:                         // > 0 if dataSmem is filled.
 563: 
 564: // Clear out per-thread counts from a previous round
 565: #pragma unroll
 566:   for (int i = 0; i < RadixSize; ++i) {
 567:     counts[i] = 0; // initialize counts to 0.
 568:   }
 569: 
 570:   // count the distribution of the bits in the radix digit at `radixDigitPos` to
 571:   // `radixDigitPos`+RADIX_BITS-1 for values that match the desired pattern
 572:   // ((val & desiredMask) == desired). counts[] will hold the results for the
 573:   // current warp.
 574:   if (dataSmemSize >
 575:       0) { // if shared memory is filled, use dataSmem as the input data.
 576:     countRadixLoop<scalar_t, bitwise_t, index_t, CountType, RadixSize, RadixBits, /*prefetch =*/ false>(
 577:         counts,
 578:         desired,
 579:         desiredMask,
 580:         radixDigitPos,
 581:         dataSmemSize,
 582:         [&](index_t i) -> scalar_t { return dataSmem[i]; });
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 583-593
```cpp
 583:   } else { // if shared memory is not filled, fall back to global memory.
 584:     countRadixLoop<scalar_t, bitwise_t, index_t, CountType, RadixSize, RadixBits, /*prefetch =*/ true>(
 585:         counts,
 586:         desired,
 587:         desiredMask,
 588:         radixDigitPos,
 589:         sliceSize,
 590:         [&](index_t i) -> scalar_t {
 591:           return doLdg(&data[i * withinSliceStride]);
 592:         });
 593:   }
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 595-595
```cpp
 595:   // aggregate counts across all warps and distribute results back to all threads.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 596-600
```cpp
 596:   countRadixAggregateCounts<CountType, RadixSize, RadixBits>(
 597:     counts,
 598:     smem,
 599:     buffer_index);
 600: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 602-605
```cpp
 602: // This is the main loop of the findPattern function that finds the unique value
 603: // that matches the pattern ((val & desired) == desiredMask) in the input data.
 604: // DataAccessor is a function that returns the input data value at index i.
 605: // It could potentially be a global memory accessor or a shared memory accessor.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 606-619
```cpp
 606: template <
 607:     typename scalar_t,
 608:     typename bitwise_t,
 609:     typename index_t,
 610:     typename DataAccessor>
 611: __device__ __forceinline__ scalar_t findPatternLoop(
 612:     scalar_t* smem, // shared memory for inter-thread communication of the found
 613:                     // value.
 614:     bitwise_t
 615:         desired, // combined with desiredMask to filter relevant elements. An
 616:                  // element is relevant if ((val & desiredMask) == desired).
 617:     bitwise_t
 618:         desiredMask, // combined with desired to filter relevant elements. An
 619:                      // element is relevant if ((val & desiredMask) == desired).
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 620-641
```cpp
 620:     index_t loopBound, // the upper bound of the loop.
 621:     DataAccessor&&
 622:         getData) { // a function that returns the input data value at index i.
 623: 
 624:   // TODO: this loop has two areas for improvement:
 625:   //   1. no need to synchronize two times at each iteration. The assumption
 626:   //   here is that the
 627:   //      data is unique. So we can have the loop truncated to the part that
 628:   //      smem is filled. We then do __syncthreads outside the loop. The current
 629:   //      early termination is probably costing us way more performance than
 630:   //      it's worth. If synchronization is moved outside the loop, we no longer
 631:   //      need to pad loopbound to round_up(loopbound, blockDim.x).
 632:   //   2. given this loop is potentially reading from global memory, we can
 633:   //   prefetch the next value
 634:   //      to improve performance. But it should not have a significant impact
 635:   //      unless point 1 above is addressed.
 636: 
 637:   // we pad loopbound to round_up(loopbound, blockDim.x) to make sure all
 638:   // threads in the block participate in the synchronization.
 639:   for (index_t i = threadIdx.x;
 640:        i < round_up(loopBound, static_cast<index_t>(blockDim.x));
 641:        i += blockDim.x) {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 642-643
```cpp
 642:     bool inRange = (i < loopBound);
 643:     scalar_t v = inRange ? getData(i) : static_cast<scalar_t>(0);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 645-652
```cpp
 645:     if (inRange &&
 646:         ((TopKTypeConfig<scalar_t>::convert(v) & desiredMask) == desired)) {
 647:       // There should not be conflicts if we are using findPattern,
 648:       // since the result is unique
 649:       smem[0] = static_cast<scalar_t>(1); // set the flag to 1.
 650:       smem[1] = v; // store the value in smem. can't use val as the flag, since
 651:                    // it could be 0.
 652:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 654-655
```cpp
 654:     __syncthreads(); // wait for all threads in the warp to finish setting the
 655:                      // flag and storing the value.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 657-658
```cpp
 657:     scalar_t found = smem[0]; // read the flag from smem.
 658:     scalar_t val = smem[1]; // read the value from smem.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 660-661
```cpp
 660:     __syncthreads(); // wait for all threads in the warp to finish reading the
 661:                      // flag and value.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 663-664
```cpp
 663:     // Checking to see if a thread found the value. If so, all threads return
 664:     // this value.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 665-668
```cpp
 665:     if (found != static_cast<scalar_t>(0)) {
 666:       return val;
 667:     }
 668:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 670-672
```cpp
 670:   CUDA_KERNEL_ASSERT(false); // should not get here.
 671:   return static_cast<scalar_t>(0); // to make sure the compiler is happy.
 672: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 674-677
```cpp
 674: // This function finds the unique value that matches the pattern
 675: // ((val & desired) == desiredMask) in the input data.
 676: // this is an smem-friendly version of the findPattern function.
 677: // It works when data is in global memory or in shared memory.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 678-691
```cpp
 678: template <typename scalar_t, typename bitwise_t, typename index_t>
 679: __device__ scalar_t findPatternDataSmem(
 680:     scalar_t* smem, // shared memory for inter-thread communication of the found
 681:                     // value.
 682:     const scalar_t* data, // input data.
 683:     index_t sliceSize, // size of the input slice.
 684:     index_t withinSliceStride, // stride of the input slice.
 685:     bitwise_t
 686:         desired, // combined with desiredMask to filter relevant elements. An
 687:                  // element is relevant if ((val & desiredMask) == desired).
 688:     bitwise_t
 689:         desiredMask, // combined with desired to filter relevant elements. An
 690:                      // element is relevant if ((val & desiredMask) == desired).
 691:     const scalar_t* dataSmem, // input data stored in shared memory.
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 692-713
```cpp
 692:     index_t dataSmemSize) { // input data size stored in shared memory.
 693: 
 694:   // Ensure all threads have finished reading from smem before overwriting it.
 695:   // countRadixAggregateCounts Stage 3 reads from smem[buffer_offset + i];
 696:   // when buffer_offset == 0, those locations overlap with smem[0]/smem[1]
 697:   // written below. Warp 0 (which writes smem[0]/smem[1]) may get ahead of
 698:   // lagging warps still in Stage 3. Syncing here (rather than at the end of
 699:   // Stage 3) is cheaper because findPatternDataSmem is called at most once per
 700:   // radixSelect invocation, only when a unique element is found (count == 1).
 701:   __syncthreads();
 702: 
 703:   // initialize smem to 0.
 704:   // smem[0] is a flag to indicate if a value has been found.
 705:   // smem[1] is the found value.
 706:   if (threadIdx.x < 2) {
 707:     smem[threadIdx.x] = static_cast<scalar_t>(0);
 708:   }
 709: 
 710:   __syncthreads(); // all threads in the block wait for smem to be initialized.
 711: 
 712:   if (dataSmemSize >
 713:       0) { // if shared memory is filled, use dataSmem as the input data.
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 714-723
```cpp
 714:     return findPatternLoop<scalar_t, bitwise_t, index_t>(
 715:         smem, desired, desiredMask, dataSmemSize, [&](index_t i) -> scalar_t {
 716:           return dataSmem[i];
 717:         });
 718:   } else { // if shared memory is not filled, fall back to global memory.
 719:     return findPatternLoop<scalar_t, bitwise_t, index_t>(
 720:         smem, desired, desiredMask, sliceSize, [&](index_t i) -> scalar_t {
 721:           return doLdg(&data[i * withinSliceStride]);
 722:         });
 723:   }
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 725-727
```cpp
 725:   return static_cast<scalar_t>(
 726:       0); // should not get here. This is to make sure the compiler is happy.
 727: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 729-749
```cpp
 729: // This function fills the shared memory dataSmem with the input data.
 730: // It is called at each iteration of the main loop of the radixSelect function.
 731: //
 732: // Four possible scenarios:
 733: //    1. dataSmem is already filled (dataSmemSize > 0). This means at a previous
 734: //    iteration
 735: //       we have filled the shared memory with the input data. We return.
 736: //    2. dataSmem is not filled (dataSmemSize == 0) and the input data is small
 737: //    enough to
 738: //       fit into shared memory (sliceSize <= dataSmemCap). If this case
 739: //       happens, it should happen at the first iteration. In this case, we put
 740: //       all the data into shared memory.
 741: //    3. dataSmem is not filled (dataSmemSize == 0) and the input data, although
 742: //    not fitting
 743: //       into shared memory originally (otherwise we would have ended up in case
 744: //       2), now fits into shared memory (dataSizeRemaining <= dataSmemCap). In
 745: //       this case, filter the data using the desired pattern ((val &
 746: //       desiredMask) == desired) and put the filtered data into shared memory.
 747: //    4. None of the above. Data does not fit into shared memory. We return. The
 748: //    situation
 749: //       may change in the next iteration.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 750-763
```cpp
 750: template <typename scalar_t, typename bitwise_t, typename index_t>
 751: __device__ __forceinline__ void fillDataSmem(
 752:     scalar_t* dataSmem, // shared memory to store the input data.
 753:     index_t
 754:         dataSmemCap, // max number of elements that can be stored in dataSmem.
 755:     index_t
 756:         dataSizeRemaining, // number of relevant elements remaining. We put data
 757:                            // on dataSmem once dataSizeRemaining <= dataSmemCap.
 758:     index_t& dataSmemSize, // actual number of elements in dataSmem.
 759:     index_t sliceSize, // size of the input slice.
 760:     index_t withinSliceStride, // stride of the input slice.
 761:     const scalar_t* data, // input data.
 762:     bitwise_t
 763:         desired, // combined with desiredMask to filter relevant elements. An
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 764-764
```cpp
 764:                  // element is relevant if ((val & desiredMask) == desired).
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 765-786
```cpp
 765:     bitwise_t
 766:         desiredMask, // combined with desired to filter relevant elements. An
 767:                      // element is relevant if ((val & desiredMask) == desired).
 768:     int& DataSmemWriteIndex // index used to write data to dataSmem. Incremented
 769:                             // atomically. Shared by all threads in the block.
 770: ) {
 771:   if (dataSmemSize > 0)
 772:     return; // already filled
 773: 
 774:   if (sliceSize <= dataSmemCap) { // if the input data is small enough, put all
 775:                                   // of it into shared memory.
 776: 
 777:     // reading from global memory. Prefetching to improve performance.
 778:     scalar_t v = static_cast<scalar_t>(0);
 779:     if (threadIdx.x < sliceSize)
 780:       v = doLdg(&data[threadIdx.x * withinSliceStride]);
 781:     for (index_t i = threadIdx.x; i < sliceSize; i += blockDim.x) {
 782:       scalar_t v_next = (i + blockDim.x) < sliceSize
 783:           ? doLdg(&data[(i + blockDim.x) * withinSliceStride])
 784:           : static_cast<scalar_t>(0);
 785:       dataSmem[i] = v;
 786:       v = v_next; // closing the prefetching loop.
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 787-787
```cpp
 787:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 789-790
```cpp
 789:     __syncthreads(); // wait for all threads in the block to finish writing to
 790:                      // dataSmem.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 792-795
```cpp
 792:     if (threadIdx.x == 0) {
 793:       dataSmemSize = sliceSize; // thread 0 updates dataSmemSize to the size of
 794:                                 // the input slice.
 795:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 797-798
```cpp
 797:     __syncthreads(); // wait for all threads in the block to see the updated
 798:                      // dataSmemSize.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 800-808
```cpp
 800:   } else if (dataSizeRemaining <= dataSmemCap) { // if data did not fit
 801:                                                  // originally, but now it does.
 802:     // if this is the case, data needs to be filtered so only the relevant data
 803:     // is stored in dataSmem. Each warp performs an internal counting of the
 804:     // number of elements that match the desired pattern. Then reserves slots in
 805:     // dataSmem for the matching elements by atomically incrementing
 806:     // DataSmemWriteIndex. Finally, each thread within the warp writes its value
 807:     // to the appropriate slot in dataSmem. This is done to minimize the amount
 808:     // of time each warp spends waiting for others.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 810-810
```cpp
 810:     int lane_id = at::cuda::getLaneId(); // = threadIdx.x % WARP_SIZE
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 812-812
```cpp
 812:     // prefetching from global memory.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 813-815
```cpp
 813:     scalar_t v = threadIdx.x < sliceSize
 814:         ? doLdg(&data[threadIdx.x * withinSliceStride])
 815:         : static_cast<scalar_t>(0);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 817-838
```cpp
 817:     for (index_t i = threadIdx.x; i < sliceSize;
 818:          i += blockDim.x) {
 819:       scalar_t v_next = (i + blockDim.x) < sliceSize
 820:           ? doLdg(&data[(i + blockDim.x) * withinSliceStride])
 821:           : static_cast<scalar_t>(0);
 822: 
 823:       bool match =
 824:           (TopKTypeConfig<scalar_t>::convert(v) & desiredMask) == desired;
 825: 
 826:       // Warp-level ballot
 827:       uint64_t ballot = WARP_BALLOT(
 828:           match); // what threads in this warp match the desired pattern?
 829:       int warp_count = __popcll(
 830:           ballot); // how many threads in this warp match the desired pattern?
 831: 
 832:       int warp_base = 0; // base index to write data to dataSmem shared by all
 833:                          // threads in the warp.
 834:       if (lane_id == 0 &&
 835:           warp_count >
 836:               0) { // warp_count > 0 means there are matching elements in this
 837:                    // warp. Only thread 0 in the warp needs to do this.
 838:         warp_base = atomicAdd(
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 839-844
```cpp
 839:             &DataSmemWriteIndex,
 840:             warp_count); // reserve warp_count slots in dataSmem for this warp,
 841:                          // and get the base index.
 842:       }
 843:       warp_base = __shfl(
 844:           warp_base, 0); // broadcast the warp_base to all threads in the warp.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 846-855
```cpp
 846:       if (match) { // if the current thread has a matching value, store the
 847:                    // value in dataSmem.
 848:         uint64_t my_mask =
 849:             (1ULL << lane_id) - 1; // a bitmask: [0, 0, 0, ..., 0, 1, 1, 1, ...,
 850:                                    // 1] with (64-lane_id) 0s and lane_id 1s.
 851:         int my_offset = __popcll(
 852:             ballot & my_mask); // count the number of threads that have matches
 853:                                // to the right of the current thread in bitmask.
 854:         dataSmem[warp_base + my_offset] = v; // store the value in dataSmem.
 855:       }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 857-858
```cpp
 857:       v = v_next; // closing the prefetching loop.
 858:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 860-861
```cpp
 860:     __syncthreads(); // wait for all threads in the block to finish writing to
 861:                      // dataSmem.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 863-866
```cpp
 863:     if (threadIdx.x == 0) {
 864:       dataSmemSize = DataSmemWriteIndex; // thread 0 updates dataSmemSize to the
 865:                                          // number of elements in dataSmem.
 866:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 868-871
```cpp
 868:     __syncthreads(); // all threads in the block wait for dataSmemSize to be
 869:                      // updated.
 870:   }
 871: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 873-873
```cpp
 873: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 875-875
```cpp
 875: // Returns the top-Kth element found in the data using radix selection
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 876-897
```cpp
 876: template <typename scalar_t, typename bitwise_t, typename index_t>
 877: __device__ void radixSelect(
 878:     const scalar_t* data,
 879:     index_t k,
 880:     bool largest,
 881:     index_t sliceSize,
 882:     index_t withinSliceStride,
 883:     index_t* smem,
 884:     scalar_t* topK) {
 885:   // Per-thread buckets into which we accumulate digit counts in our
 886:   // radix
 887:   //
 888:   // counts must be index_t to safely handle sliceSize > INT_MAX.
 889:   index_t counts[RADIX_SIZE];
 890: 
 891: #ifdef USE_ROCM
 892: 
 893:   // this kernel reads all the data at most (sizeof(scalar_t)*2/RADIX_BITS + 1)
 894:   // times. if data fits into shared memory, we can avoid reading data from
 895:   // global memory. if not, we may still be able to put the filtered data, after
 896:   // a few iterations, into shared memory. after every pass, relevant data is
 897:   // likely reduced by a factor of RADIX_SIZE. dataSmem is used to store the
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `radixSelect`.
- CN: 该代码块定义或继续实现 `radixSelect`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 898-898
```cpp
 898:   // relevant data.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 899-910
```cpp
 899:   constexpr index_t DATA_SMEM_BYTES = 3 *
 900:       1024; // 3KB is a good compromise between memory usage and performance.
 901:   constexpr index_t dataSmemCap =
 902:       DATA_SMEM_BYTES /
 903:       sizeof(
 904:           scalar_t); // max number of elements that can be stored in dataSmem.
 905:   __shared__ scalar_t dataSmem[dataSmemCap];
 906:   __shared__ index_t dataSmemSize; // actual number of elements in dataSmem.
 907:   __shared__ index_t
 908:       dataSizeRemaining; // number of relevant elements remaining. We put data
 909:                          // on dataSmem once dataSizeRemaining <= dataSmemCap.
 910:   __shared__ int DataSmemWriteIndex; // index used to write data to dataSmem.
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 912-916
```cpp
 912:   if (threadIdx.x == 0) {
 913:     dataSmemSize = 0;
 914:     DataSmemWriteIndex = 0;
 915:     dataSizeRemaining = sliceSize;
 916:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 918-919
```cpp
 918:   __syncthreads(); // so the initialization is visible to all threads in the
 919:                    // blocks.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 921-926
```cpp
 921:   // buffer index for smem. We use two segments of smem for inter-warp communication of counts.
 922:   // Given the counting operation in countRadixUsingMaskDataSmem performs __syncthreads() internally,
 923:   // we need to alternate between the at most two segments of smem to avoid race conditions.
 924:   // No more than two iterations of the loop will be "in flight" at any given time because
 925:   // of the __syncthreads() in countRadixUsingMaskDataSmem.
 926:   // buffer_index is either 0 or 1. It is toggled after each countRadixUsingMaskDataSmem invocation.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 927-927
```cpp
 927:   int buffer_index = 0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 929-929
```cpp
 929: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 931-933
```cpp
 931:   // We only consider elements x such that (x & desiredMask) == desired
 932:   // Initially, we consider all elements of the array, so the above
 933:   // statement is true regardless of input.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 934-935
```cpp
 934:   bitwise_t desired = 0;
 935:   bitwise_t desiredMask = 0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 937-939
```cpp
 937:   // We are looking for the top kToFind-th element when iterating over
 938:   // digits; this count gets reduced by elimination when counting
 939:   // successive digits
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 940-940
```cpp
 940:   index_t kToFind = k;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 942-943
```cpp
 942:   // We start at the most significant digit in our radix, scanning
 943:   // through to the least significant digit
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 944-965
```cpp
 944:   for (int digitPos = sizeof(scalar_t) * 8 - RADIX_BITS; digitPos >= 0;
 945:        digitPos -= RADIX_BITS) {
 946:     // Count radix distribution for the current position and reduce
 947:     // across all threads
 948: 
 949: #ifdef USE_ROCM
 950: 
 951:     // fill dataSmem with the input data if not already filled.
 952:     fillDataSmem<scalar_t, bitwise_t, index_t>(
 953:         dataSmem,
 954:         dataSmemCap,
 955:         dataSizeRemaining,
 956:         dataSmemSize,
 957:         sliceSize,
 958:         withinSliceStride,
 959:         data,
 960:         desired,
 961:         desiredMask,
 962:         DataSmemWriteIndex);
 963: 
 964:     // count the distribution of the bits in the radix digit at `digitPos` to
 965:     // `digitPos`+RADIX_BITS-1
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 966-979
```cpp
 966:     countRadixUsingMaskDataSmem<
 967:         scalar_t,
 968:         bitwise_t,
 969:         index_t,
 970:         index_t,
 971:         RADIX_SIZE,
 972:         RADIX_BITS>(
 973:         counts,
 974:         smem,
 975:         buffer_index,
 976:         desired,
 977:         desiredMask,
 978:         digitPos,
 979:         sliceSize,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 980-983
```cpp
 980:         withinSliceStride,
 981:         data,
 982:         dataSmem,
 983:         dataSmemSize);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 985-985
```cpp
 985:     buffer_index ^= 1; // toggle buffer index.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 987-987
```cpp
 987: #else
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 988-1001
```cpp
 988:     countRadixUsingMask<
 989:         scalar_t,
 990:         bitwise_t,
 991:         index_t,
 992:         index_t,
 993:         RADIX_SIZE,
 994:         RADIX_BITS>(
 995:         counts,
 996:         smem,
 997:         desired,
 998:         desiredMask,
 999:         digitPos,
1000:         sliceSize,
1001:         withinSliceStride,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1002-1002
```cpp
1002:         data);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1004-1004
```cpp
1004: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1005-1026
```cpp
1005:     auto found_unique = [&](int i, index_t count) -> bool {
1006:       /* All threads have the same value in counts here, so all */
1007:       /* threads will return from the function. */
1008:       if (count == 1 && kToFind == 1) {
1009:         /* There is a unique answer. */
1010:         desired = at::cuda::Bitfield<bitwise_t>::setBitfield(
1011:             desired, i, digitPos, RADIX_BITS);
1012:         desiredMask = at::cuda::Bitfield<bitwise_t>::setBitfield(
1013:             desiredMask, RADIX_MASK, digitPos, RADIX_BITS);
1014: 
1015:         /* The answer is now the unique element v such that: */
1016:         /* (v & desiredMask) == desired */
1017:         /* However, we do not yet know what the actual element is. We */
1018:         /* need to perform a search through the data to find the */
1019:         /* element that matches this pattern. */
1020: 
1021: #ifndef USE_ROCM
1022: 
1023:         *topK = findPattern<scalar_t, bitwise_t, index_t>(
1024:             (scalar_t*)smem,
1025:             data,
1026:             sliceSize,
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1027-1029
```cpp
1027:             withinSliceStride,
1028:             desired,
1029:             desiredMask);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1031-1031
```cpp
1031: #else
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1032-1033
```cpp
1032:         // find the unique value that matches the desired pattern
1033:         *topK = findPatternDataSmem<scalar_t, bitwise_t, index_t>(
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1034-1047
```cpp
1034:             (scalar_t*)smem,
1035:             data,
1036:             sliceSize,
1037:             withinSliceStride,
1038:             desired,
1039:             desiredMask,
1040:             dataSmem,
1041:             dataSmemSize);
1042: #endif
1043:         return true;
1044:       }
1045:       return false;
1046:     };
1047:     auto found_non_unique = [&](int i, index_t count) -> bool {
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1048-1069
```cpp
1048:       if (count >= kToFind) {
1049:         desired = at::cuda::Bitfield<bitwise_t>::setBitfield(
1050:             desired, i, digitPos, RADIX_BITS);
1051:         desiredMask = at::cuda::Bitfield<bitwise_t>::setBitfield(
1052:             desiredMask, RADIX_MASK, digitPos, RADIX_BITS);
1053: 
1054: #ifdef USE_ROCM
1055:         if (dataSmemSize == 0) { // we only care about updating
1056:                                  // dataSizeRemaining when dataSmem is empty.
1057:           if (threadIdx.x == 0) {
1058:             // this bucket has count >= kToFind elements. This means topK is in
1059:             // this bucket and the number of elements with value & desiredMask
1060:             // == desired (which is the relevant data) equals count. so we
1061:             // update dataSizeRemaining to count.
1062:             dataSizeRemaining = count;
1063:           }
1064:           __syncthreads();
1065:         }
1066: #endif
1067:         /* The top-Kth element v must now be one such that: */
1068:         /* (v & desiredMask == desired) */
1069:         /* but we haven't narrowed it down; we must check the next */
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1070-1070
```cpp
1070:         /* least-significant digit */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1071-1075
```cpp
1071:         return true;
1072:       }
1073:       kToFind -= count;
1074:       return false; // continue the loop
1075:     };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1077-1078
```cpp
1077:     // All threads participate in the comparisons below to know the
1078:     // final result
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1079-1100
```cpp
1079:     if (largest) {
1080:       // Process in descending order
1081: #pragma unroll
1082:       for (int i = RADIX_SIZE - 1; i >= 0; --i) {
1083:         index_t count = counts[i];
1084:         if (found_unique(i, count)) {
1085:           return;
1086:         }
1087:         if (found_non_unique(i, count)) {
1088:           break;
1089:         }
1090:       }
1091:     } else {
1092:       // Process in ascending order
1093: #pragma unroll
1094:       for (int i = 0; i < RADIX_SIZE; ++i) {
1095:         index_t count = counts[i];
1096:         if (found_unique(i, count)) {
1097:           return;
1098:         }
1099:         if (found_non_unique(i, count)) {
1100:           break;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1101-1104
```cpp
1101:         }
1102:       }
1103:     }
1104:   } // end digitPos for
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1106-1108
```cpp
1106:   // There is no unique result, but there is a non-unique result
1107:   // matching `desired` exactly
1108:   *topK = TopKTypeConfig<scalar_t>::deconvert(desired);
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1109-1110
```cpp
1109: }
1110: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/ceil_div.h>`
  - `<c10/macros/Macros.h>`
  - `<ATen/cuda/AsmUtils.cuh>`
  - `<ATen/cuda/Atomic.cuh>`
  - `<ATen/cuda/DeviceUtils.cuh>`
  - `<type_traits>`
- Runtime symbols / 运行时符号:
  - `at::cuda::Bitfield`
  - `at::cuda::getLaneId`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
