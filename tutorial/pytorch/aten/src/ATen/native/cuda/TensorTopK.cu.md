# TensorTopK.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/TensorTopK.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `writeResult`, `warpMergeSortTopK`, `constexpr`, `launch`.
- 用途（中文）: 实现与 `writeResult`, `warpMergeSortTopK`, `constexpr`, `launch` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_NO_OPERATORS
   2: #include <ATen/native/cuda/TensorTopK.h>
   3: #include <ATen/core/TensorBase.h>
   4: #include <ATen/ceil_div.h>
   5: #include <ATen/Dispatch.h>
   6: #include <c10/macros/Macros.h>
   7: #include <ATen/cuda/CUDAContext.h>
   8: #include <ATen/cuda/detail/TensorInfo.cuh>
   9: #include <ATen/cuda/detail/OffsetCalculator.cuh>
  10: #include <ATen/cuda/ScanUtils.cuh>
  11: #include <ATen/cuda/AsmUtils.cuh>
  12: #include <ATen/cuda/DeviceUtils.cuh>
  13: #include <ATen/native/cuda/SortingCommon.cuh>
  14: #include <ATen/native/cuda/SortingRadixSelect.cuh>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/TensorTopK.h>`, `<ATen/core/TensorBase.h>`, `<ATen/ceil_div.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/TensorTopK.h>`, `<ATen/core/TensorBase.h>`, `<ATen/ceil_div.h>`。

### Lines 15-23
```cpp
  15: #include <ATen/native/cuda/SortUtils.cuh>
  16: #include <ATen/native/StridedRandomAccessor.h>
  17: #include <ATen/cuda/cub.cuh>
  18: #include <c10/cuda/CUDACachingAllocator.h>
  19: #include <ATen/cuda/detail/KernelUtils.h>
  20: 
  21: #if defined(USE_ROCM)
  22: #include <rocprim/block/block_scan.hpp>
  23: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/SortUtils.cuh>`, `<ATen/native/StridedRandomAccessor.h>`, `<ATen/cuda/cub.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/SortUtils.cuh>`, `<ATen/native/StridedRandomAccessor.h>`, `<ATen/cuda/cub.cuh>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 25-25
```cpp
  25: using namespace at::native;
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 27-48
```cpp
  27: namespace at::native {
  28: 
  29: namespace sbtopk { // single_block_topk
  30: 
  31: template <typename T>
  32: struct AddOp {
  33:   __device__ __forceinline__ T operator()(T const &lhs, T const &rhs) {
  34:     return (lhs + rhs);
  35:   }
  36: };
  37: 
  38: #ifndef USE_ROCM
  39: 
  40: template <typename T, typename IndexType, int Dim, bool WithKthValues>
  41: C10_LAUNCH_BOUNDS_1(1024)
  42: __global__ void gatherTopK(at::cuda::detail::TensorInfo<const T, IndexType> input,
  43:                            IndexType inputSliceSize,
  44:                            IndexType outputSliceSize, // aka `k`
  45:                            bool largest,
  46: 
  47:                            IndexType numInputSlices,
  48:                            IndexType inputWithinSliceStride,
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `gatherTopK`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `gatherTopK`，它们会直接在 CUDA 线程上执行。

### Lines 50-51
```cpp
  50:                            at::cuda::detail::TensorInfo<T, IndexType> topK,
  51:                            IndexType topKWithinSliceStride,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 53-74
```cpp
  53:                            at::cuda::detail::TensorInfo<int64_t, IndexType> indices,
  54:                            IndexType indicesWithinSliceStride,
  55:                            T* kthValues) {
  56:   // smem is used for:
  57:   // 1. radixSelect: radix bin counts (RADIX_SIZE=4 elements)
  58:   // 2. exclusiveBinaryPrefixScan: warp prefix sums (≤32 elements)
  59:   // 3. findPattern: flag and value (2 elements, cast to scalar_t*)
  60:   //
  61:   // Type must be IndexType to safely handle sliceSize > INT_MAX.
  62:   // In radix selection, counts can exceed INT_MAX when billions of
  63:   // elements fall into a single radix bin.
  64:   __shared__ IndexType smem[32];
  65:   IndexType slice = getLinearBlockId<IndexType>();
  66:   if (slice >= numInputSlices) {
  67:     return;
  68:   }
  69: 
  70:   // Find the start offset for our slice
  71:   IndexType sliceStartIndex =
  72:     at::cuda::detail::IndexToOffset<const T, IndexType, Dim>::get(slice, input);
  73:   IndexType topKSliceStartIndex =
  74:     at::cuda::detail::IndexToOffset<T, IndexType, Dim>::get(slice, topK);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 75-76
```cpp
  75:   IndexType indicesSliceStartIndex =
  76:     at::cuda::detail::IndexToOffset<int64_t, IndexType, Dim>::get(slice, indices);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 78-80
```cpp
  78:   const T* inputSliceStart = &input.data[sliceStartIndex];
  79:   T* topKSliceStart = &topK.data[topKSliceStartIndex];
  80:   int64_t* indicesSliceStart = &indices.data[indicesSliceStartIndex];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 82-82
```cpp
  82:   // Find the k-th highest element in our input
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 83-93
```cpp
  83:   T topKValue;
  84:   if (WithKthValues){
  85:     topKValue = kthValues[slice];
  86:   } else {
  87:     topKValue = static_cast<T>(0);
  88:     radixSelect<T, typename TopKTypeConfig<T>::RadixType, IndexType>(
  89:       inputSliceStart, outputSliceSize, largest,
  90:       inputSliceSize, inputWithinSliceStride,
  91:       smem, &topKValue);
  92:   }
  93:   const auto topKConverted = at::native::TopKTypeConfig<T>::convert(topKValue);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 95-108
```cpp
  95:   // Every value that is strictly less/greater than `pattern`
  96:   // (depending on sort dir) in sorted int format is in the top-K.
  97:   // The top-K value itself might not be unique.
  98:   //
  99:   // Since there are a variable number of elements that we see that
 100:   // are within the top-k, we don't know at what index to write out
 101:   // the resulting values.
 102:   // In order to get this, we perform an exclusive prefix sum of
 103:   // `hasTopK`. This will return the resulting index into which we
 104:   // need to write the result, if a thread has a result.
 105: 
 106:   // All threads need to participate in the loop and the prefix sum,
 107:   // but not necessarily in the load; hence loop bounds being rounded
 108:   // up to a multiple of the block dim.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 109-110
```cpp
 109:   IndexType numIterations = round_up(inputSliceSize, (IndexType) blockDim.x);
 110:   IndexType writeIndexStart = 0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 112-133
```cpp
 112:   for (IndexType i = threadIdx.x; i < numIterations; i += blockDim.x) {
 113:     bool inRange = (i < inputSliceSize);
 114:     T v =
 115:       inRange ? doLdg(&inputSliceStart[i * inputWithinSliceStride]) : static_cast<T>(0);
 116:     const auto convertedV = at::native::TopKTypeConfig<T>::convert(v);
 117:     bool hasTopK;
 118:     if (largest) {
 119:       hasTopK = inRange && (convertedV > topKConverted);
 120:     } else {
 121:       hasTopK = inRange && (convertedV < topKConverted);
 122:     }
 123: 
 124:     IndexType index;
 125:     IndexType carry;
 126:     at::cuda::exclusiveBinaryPrefixScan<IndexType, true>(
 127:         smem, hasTopK, &index, &carry, AddOp<IndexType>());
 128: 
 129:     if (hasTopK) {
 130:       IndexType writeIndex = writeIndexStart + index;
 131:       CUDA_KERNEL_ASSERT(writeIndex < outputSliceSize);
 132: 
 133:       IndexType topKOffset = writeIndex * topKWithinSliceStride;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 134-134
```cpp
 134:       IndexType indexOffset = writeIndex * indicesWithinSliceStride;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 136-138
```cpp
 136:       topKSliceStart[topKOffset] = v;
 137:       indicesSliceStart[indexOffset] = i;
 138:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 140-141
```cpp
 140:     writeIndexStart += carry;
 141:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 143-147
```cpp
 143:   // We need to fill in the rest with actual == top-K values.
 144:   // The number that we need is outputSliceSize -
 145:   // writeIndexStart. There might be more than that number available,
 146:   // in which case we have to choose the first seen set. We do this
 147:   // via a prefix sum to calculate indices for writing results.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 148-149
```cpp
 148:   CUDA_KERNEL_ASSERT(outputSliceSize >= writeIndexStart);
 149:   IndexType topKRemaining = (outputSliceSize - writeIndexStart);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 151-172
```cpp
 151:   for (IndexType i = threadIdx.x; i < numIterations; i += blockDim.x) {
 152:     bool inRange = (i < inputSliceSize);
 153:     T v =
 154:       inRange ? doLdg(&inputSliceStart[i * inputWithinSliceStride]) : static_cast<T>(0);
 155:     const auto convertedV = at::native::TopKTypeConfig<T>::convert(v);
 156:     bool hasTopK = inRange && (convertedV == topKConverted);
 157: 
 158:     IndexType index;
 159:     IndexType carry;
 160:     at::cuda::exclusiveBinaryPrefixScan<IndexType, true>(
 161:         smem, hasTopK, &index, &carry, AddOp<IndexType>());
 162: 
 163:     if (hasTopK && index < topKRemaining) {
 164:       IndexType writeIndex = writeIndexStart + index;
 165:       CUDA_KERNEL_ASSERT(writeIndex < outputSliceSize);
 166: 
 167:       IndexType topKOffset = writeIndex * topKWithinSliceStride;
 168:       IndexType indexOffset = writeIndex * indicesWithinSliceStride;
 169: 
 170:       topKSliceStart[topKOffset] = v;
 171:       indicesSliceStart[indexOffset] = i;
 172:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 174-176
```cpp
 174:     if (carry >= topKRemaining) {
 175:       break;
 176:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 178-180
```cpp
 178:     topKRemaining -= carry;
 179:     writeIndexStart += carry;
 180:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 182-182
```cpp
 182: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 184-184
```cpp
 184: #else
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 186-186
```cpp
 186: /*
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 187-200
```cpp
 187: This implementation of gatherTopK is a modified version of the original gatherTopK kernel. This kernel is called
 188: after we have found the k-th highest (or lowest, depending on the sort direction) element in our input. It gathers
 189: values that are greater (or less than) than the k-th element (phase 1) and then adds the values that are equal to
 190: the k-th element as long as there is space available (phase 2). In the original implementation, we call
 191: exclusiveBinaryPrefixScan to calculate the index to write the result to. However, exclusiveBinaryPrefixScan has two
 192: block level synchronization points, which is not efficient, specially considering that exclusiveBinaryPrefixScan is
 193: called in a loop. In this implementation, we use warp level compaction to calculate the index to write the result
 194: to. In both phases, each warp first counts the number of values it intends to add to the result. Then through an
 195: atomic add, the warp reserves space for itself (atomically increases the write index variable) and then writes the
 196: result to the corresponding indices. This requires no block level synchronization. It should be noted that we have
 197: added a block level synchronization point after phase 1 to make sure all threads have completed phase 1. This
 198: synchronization is cheaper than the ones in exclusiveBinaryPrefixScan because it is called only once. This
 199: synchronization is necessary because phase 1 assumes it always has space to write all the values that are larger (or
 200: smaller) than the k-th element but phase 2 tops off the output as long as there is space available.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 201-208
```cpp
 201: */
 202: 
 203: // helper function to reserve space for a warp in the output.
 204: // hasTopK: boolean flag to indicate if the current thread has a value to add to the output.
 205: // writeIndexStart: atomic variable to track the index to write the result to.
 206: // start_index: index to write the result to. (output of function)
 207: // my_offset: offset to write the result to. (output of function)
 208: // warp_count: number of threads that have values to add to the output. (output of function)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 209-229
```cpp
 209: template <typename IndexType>
 210: __device__ __forceinline__ void reserveWarpSpace(bool hasTopK,
 211:                                                 IndexType& writeIndexStart,
 212:                                                 IndexType& start_index,
 213:                                                 int& my_offset,
 214:                                                 int& warp_count) {
 215:   auto ballot = WARP_BALLOT(hasTopK); // a bitmask of threads that have hasTopK == true within the warp.
 216:   warp_count = __popcll(ballot); // count the number of threads that have hasTopK == true within the warp.
 217: 
 218:   int lane_id = at::cuda::getLaneId();
 219: 
 220:   // if > 0 threads have hasTopK == true within the warp,
 221:   // reserve space for them by incrementing writeIndexStart atomically + saving the old value  as start index.
 222:   if (warp_count > 0 && lane_id == 0) {
 223:     start_index = atomicAdd(&writeIndexStart, (IndexType)warp_count);
 224:   }
 225:   start_index = __shfl(start_index, 0); // broadcast the start index to all threads in the warp.
 226: 
 227:   uint64_t mask = (1ULL << lane_id) - 1; // a bitmask: [0, 0, 0, ..., 0, 1, 1, 1, ..., 1] with (64-lane_id) 0s and (lane_id) 1s
 228:   my_offset = __popcll(ballot & mask);  // get number of threads that have hasTopK == true to the right of the current lane
 229: }
```
- EN: This block defines or continues the implementation of `reserveWarpSpace`.
- CN: 该代码块定义或继续实现 `reserveWarpSpace`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 231-231
```cpp
 231: // helper function to write the result to the output.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 232-246
```cpp
 232: template <typename T, typename IndexType>
 233: __device__ __forceinline__ void writeResult(T* topKSliceStart,
 234:                                             int64_t* indicesSliceStart,
 235:                                             IndexType topKWithinSliceStride,
 236:                                             IndexType indicesWithinSliceStride,
 237:                                             IndexType outputSliceSize,
 238:                                             IndexType writeIndex,
 239:                                             T v,
 240:                                             IndexType i){
 241:   CUDA_KERNEL_ASSERT(writeIndex < outputSliceSize); // assert that the write index is within the output slice size.
 242:   IndexType topKOffset = writeIndex * topKWithinSliceStride; // calculate the offset to the topk value in the output slice.
 243:   IndexType indexOffset = writeIndex * indicesWithinSliceStride; // calculate the offset to the index in the output slice.
 244:   topKSliceStart[topKOffset] = v; // write the value to the output slice.
 245:   indicesSliceStart[indexOffset] = i; // write the index to the output slice.
 246: }
```
- EN: This block defines or continues the implementation of `writeResult`.
- CN: 该代码块定义或继续实现 `writeResult`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 248-253
```cpp
 248: template <typename T, typename IndexType, int Dim, bool WithKthValues>
 249: C10_LAUNCH_BOUNDS_1(1024)
 250: __global__ void gatherTopK(at::cuda::detail::TensorInfo<const T, IndexType> input,
 251:                             IndexType inputSliceSize,
 252:                             IndexType outputSliceSize, // aka `k`
 253:                             bool largest,
```
- EN: This block defines GPU kernel entry point(s) `gatherTopK`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `gatherTopK`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 255-256
```cpp
 255:                             IndexType numInputSlices,
 256:                             IndexType inputWithinSliceStride,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 258-259
```cpp
 258:                             at::cuda::detail::TensorInfo<T, IndexType> topK,
 259:                             IndexType topKWithinSliceStride,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 261-282
```cpp
 261:                             at::cuda::detail::TensorInfo<int64_t, IndexType> indices,
 262:                             IndexType indicesWithinSliceStride,
 263:                             T* kthValues) {
 264: 
 265:   // smem and counts must use IndexType to safely handle sliceSize > INT_MAX.
 266:   // In radix selection, counts tracks elements matching a radix pattern,
 267:   // which can exceed INT_MAX when billions of elements fall into one bin.
 268: 
 269:   // Maximum shared memory size for radix select (used in countRadixAggregateCounts): NUM_BUFFERS * MAX_WARPS * RADIX_SIZE.
 270:   // HIP workgroups have at most 1024 threads. Warp size is at least 32 (can be 64 on some
 271:   // architectures), so we use 32 for safety: 2 buffers * (1024/32) warps * 4 radix bins = 256.
 272:   __shared__ IndexType smem[256];
 273:   __shared__ IndexType writeIndexStart; // index to track where to write results. This is shared by all threads in the block. Increases atomically.
 274: 
 275:   IndexType slice = getLinearBlockId<IndexType>();
 276:   if (slice >= numInputSlices) {
 277:     return;
 278:   }
 279: 
 280:   // Find the start offset for our slice
 281:   IndexType sliceStartIndex =
 282:     at::cuda::detail::IndexToOffset<const T, IndexType, Dim>::get(slice, input);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 283-286
```cpp
 283:   IndexType topKSliceStartIndex =
 284:     at::cuda::detail::IndexToOffset<T, IndexType, Dim>::get(slice, topK);
 285:   IndexType indicesSliceStartIndex =
 286:     at::cuda::detail::IndexToOffset<int64_t, IndexType, Dim>::get(slice, indices);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 288-290
```cpp
 288:   const T* inputSliceStart = &input.data[sliceStartIndex];
 289:   T* topKSliceStart = &topK.data[topKSliceStartIndex];
 290:   int64_t* indicesSliceStart = &indices.data[indicesSliceStartIndex];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 292-292
```cpp
 292:   // Find the k-th highest element in our input
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 293-303
```cpp
 293:   T topKValue;
 294:   if (WithKthValues){
 295:     topKValue = kthValues[slice];
 296:   } else {
 297:     topKValue = static_cast<T>(0);
 298:     radixSelect<T, typename TopKTypeConfig<T>::RadixType, IndexType>(
 299:       inputSliceStart, outputSliceSize, largest,
 300:       inputSliceSize, inputWithinSliceStride,
 301:       smem, &topKValue);
 302:   }
 303:   const auto topKConverted = at::native::TopKTypeConfig<T>::convert(topKValue);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 305-316
```cpp
 305:   // Every value that is strictly less/greater than `pattern`
 306:   // (depending on sort dir) in sorted int format is in the top-K.
 307:   // The top-K value itself might not be unique.
 308:   //
 309:   // Since there are a variable number of elements that we see that
 310:   // are within the top-k, we don't know at what index to write out
 311:   // the resulting values.
 312:   // In order to get this, we perform warp level compaction.
 313:   // each warp counts its own number of hasTopk threads and
 314:   // reserves space for them by incrementing writeIndexStart atomically + saving the old value as start index.
 315: 
 316:   // Initialize writeIndexStart to 0 by the first thread in the block.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 317-322
```cpp
 317:   if (threadIdx.x == 0) {
 318:     writeIndexStart = 0;
 319:   }
 320:   __syncthreads();
 321:   // All threads within the warp need to participate in the loop, so rounding up to a multiple of the warp size.
 322:   IndexType numIterations = round_up(inputSliceSize, (IndexType) C10_WARP_SIZE);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 324-325
```cpp
 324:   // phase 1: write actual > `pattern` (or < `pattern`, depending on the sort direction) values to the output.
 325:   // prefetching data from global memory.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 326-347
```cpp
 326:   T v = (threadIdx.x < inputSliceSize) ? doLdg(&inputSliceStart[threadIdx.x * inputWithinSliceStride]) : static_cast<T>(0);
 327:   for (IndexType i = threadIdx.x; i < numIterations; i += blockDim.x) {
 328:     T v_next = (i + blockDim.x < inputSliceSize) ? doLdg(&inputSliceStart[(i + blockDim.x) * inputWithinSliceStride]) : static_cast<T>(0);
 329: 
 330:     bool hasTopK = false;
 331:     if (i < inputSliceSize) {
 332:       const auto convertedV = at::native::TopKTypeConfig<T>::convert(v);
 333:       hasTopK = (largest) ? (convertedV > topKConverted) : (convertedV < topKConverted);
 334:     }
 335: 
 336:     IndexType start_index;
 337:     int my_offset, warp_count;
 338:     reserveWarpSpace(hasTopK, writeIndexStart, start_index, my_offset, warp_count);
 339: 
 340:     // now warp has reserved space for itself. If hasTopK == true, we need to find the index to write the result to.
 341:     if (hasTopK) {
 342:       writeResult(topKSliceStart,
 343:         indicesSliceStart,
 344:         topKWithinSliceStride,
 345:         indicesWithinSliceStride,
 346:         outputSliceSize,
 347:         /*writeIndex=*/start_index + my_offset,
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 348-349
```cpp
 348:         /*value=*/v,
 349:         /*index=*/i);
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 350-350
```cpp
 350:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 352-353
```cpp
 352:     v = v_next;
 353:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 355-356
```cpp
 355:   // till this point, actual > `pattern` values were being written.
 356:   // we first need to sync to make sure all threads have completed phase 1:
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 357-357
```cpp
 357:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 359-366
```cpp
 359:   // We need to fill in the rest with actual == top-K values.
 360:   // The number that we need is outputSliceSize - writeIndexStart.
 361:   // There might be more than that number available in input,
 362:   // in which case we have to choose the first seen set. We do this
 363:   // in a similar warp level compaction fashion as in phase 1.
 364: 
 365:   // phase 2: write actual == `pattern` values to the output.
 366:   // prefetching data from global memory.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 367-388
```cpp
 367:   T V = (threadIdx.x < inputSliceSize) ? doLdg(&inputSliceStart[threadIdx.x * inputWithinSliceStride]) : static_cast<T>(0);
 368:   for (IndexType i = threadIdx.x; i < numIterations; i += blockDim.x) {
 369:     T V_next = (i + blockDim.x < inputSliceSize) ? doLdg(&inputSliceStart[(i + blockDim.x) * inputWithinSliceStride]) : static_cast<T>(0);
 370:     bool hasTopK = false;
 371:     if (i < inputSliceSize) {
 372:       const auto convertedV = at::native::TopKTypeConfig<T>::convert(V);
 373:       hasTopK = convertedV == topKConverted;
 374:     }
 375: 
 376:     IndexType start_index;
 377:     int my_offset, warp_count;
 378:     reserveWarpSpace(hasTopK, writeIndexStart, start_index, my_offset, warp_count);
 379: 
 380:     if ((warp_count > 0) && (outputSliceSize <= start_index)){
 381:       break; // there is no space to add topk values. Break out of the loop.
 382:     }
 383: 
 384:     if (hasTopK){
 385:       IndexType slots_available = outputSliceSize - start_index;
 386:       if (my_offset < slots_available){
 387:         writeResult(topKSliceStart,
 388:           indicesSliceStart,
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 389-396
```cpp
 389:           topKWithinSliceStride,
 390:           indicesWithinSliceStride,
 391:           outputSliceSize,
 392:           /*writeIndex=*/start_index + my_offset,
 393:           /*value=*/V,
 394:           /*index=*/i);
 395:       }
 396:     }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 398-400
```cpp
 398:     V = V_next;
 399:   }
 400: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 402-402
```cpp
 402: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 404-409
```cpp
 404: template <typename T, typename IndexType, int Dim>
 405: void launch(
 406:     at::cuda::detail::TensorInfo<const T, IndexType> input,
 407:     IndexType inputSliceSize,
 408:     IndexType outputSliceSize, // aka `k`
 409:     bool largest,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 411-412
```cpp
 411:     IndexType numInputSlices,
 412:     IndexType inputWithinSliceStride,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 414-415
```cpp
 414:     at::cuda::detail::TensorInfo<T, IndexType> topK,
 415:     IndexType topKWithinSliceStride,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 417-437
```cpp
 417:     at::cuda::detail::TensorInfo<int64_t, IndexType> indices,
 418:     IndexType indicesWithinSliceStride) {
 419: 
 420:     dim3 grid;
 421:     TORCH_INTERNAL_ASSERT(getGridFromTiles(numInputSlices, grid), "Too many slices for topk");
 422:     int warp_size = at::cuda::warp_size();
 423:     dim3 block(std::min(at::ceil_div((int64_t)inputSliceSize, (int64_t)warp_size) * (int64_t)warp_size, (int64_t)1024));
 424:     gatherTopK<T, IndexType, Dim, /* WithKthValues= */false><<<grid, block, 0, c10::cuda::getCurrentCUDAStream()>>>(
 425:         input,
 426:         inputSliceSize,
 427:         outputSliceSize,
 428:         largest,
 429:         numInputSlices,
 430:         inputWithinSliceStride,
 431:         topK,
 432:         topKWithinSliceStride,
 433:         indices,
 434:         indicesWithinSliceStride,
 435:         nullptr);
 436:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 437: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 438-438
```cpp
 438: } // namespace sbtopk
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 440-440
```cpp
 440: #if defined(USE_ROCM) && HAS_WARP_MERGE_SORT()
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 441-462
```cpp
 441: namespace warptopk {
 442: 
 443: constexpr int MAX_WARP_TOPK_SLICE = 512;
 444: 
 445: // Comparator for sorting with TopK semantics
 446: // Note: For WarpMergeSort (comparison-based sorting), we use simple comparison operators
 447: // GTOp/LTOp instead of bitwise conversion. Bitwise conversion is only needed for radix sorting.
 448: 
 449: // Kernel using WarpMergeSort for small topK operations
 450: // See Note [warp merge sort WARP_SIZE template param]
 451: template <int KeyDims, int ValueDims, int sort_size, int max_block_dim_y,
 452:           typename scalar_t, typename IndexType, bool is_descending, int WARP_SIZE>
 453: __global__ void warpMergeSortTopK(
 454:     at::cuda::detail::TensorInfo<const scalar_t, IndexType> input,
 455:     IndexType inputSliceSize,
 456:     IndexType k,
 457:     IndexType numInputSlices,
 458:     IndexType inputWithinSliceStride,
 459:     at::cuda::detail::TensorInfo<scalar_t, IndexType> topK,
 460:     IndexType topKWithinSliceStride,
 461:     at::cuda::detail::TensorInfo<int64_t, IndexType> indices,
 462:     IndexType indicesWithinSliceStride) {
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `warpMergeSortTopK`, `WarpMergeSort`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `warpMergeSortTopK`, `WarpMergeSort`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 464-464
```cpp
 464:   // Find the slice this warp is working on
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 465-466
```cpp
 465:   const IndexType blockIndex = getLinearBlockId<IndexType>();
 466:   const IndexType linearIndex = blockIndex * blockDim.y + threadIdx.y;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 468-470
```cpp
 468:   if (linearIndex >= numInputSlices) {
 469:     return;
 470:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 472-477
```cpp
 472:   const IndexType inputStartOffset =
 473:       at::cuda::detail::IndexToOffset<const scalar_t, IndexType, KeyDims>::get(linearIndex, input);
 474:   const IndexType topKStartOffset =
 475:       at::cuda::detail::IndexToOffset<scalar_t, IndexType, ValueDims>::get(linearIndex, topK);
 476:   const IndexType indicesStartOffset =
 477:       at::cuda::detail::IndexToOffset<int64_t, IndexType, ValueDims>::get(linearIndex, indices);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 479-481
```cpp
 479:   const scalar_t* input_slice = &input.data[inputStartOffset];
 480:   scalar_t* topK_slice = &topK.data[topKStartOffset];
 481:   int64_t* indices_slice = &indices.data[indicesStartOffset];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 483-483
```cpp
 483:   // Create strided accessors
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 484-486
```cpp
 484:   StridedRandomAccessor<const scalar_t, IndexType> input_iter(input_slice, inputWithinSliceStride);
 485:   StridedRandomAccessor<scalar_t, IndexType> topK_iter(topK_slice, topKWithinSliceStride);
 486:   StridedRandomAccessor<int64_t, IndexType> indices_iter(indices_slice, indicesWithinSliceStride);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 488-488
```cpp
 488:   namespace cub = ROCM_HIPCUB(at_cuda_detail::cub);
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 490-494
```cpp
 490:   CUDA_KERNEL_ASSERT(blockDim.x == WARP_SIZE);
 491:   CUDA_KERNEL_ASSERT(blockDim.y <= max_block_dim_y);
 492:   constexpr int items_per_thread = sort_size / WARP_SIZE;
 493:   static_assert(items_per_thread * WARP_SIZE == sort_size,
 494:                 "sort_size must be a multiple of WARP_SIZE template param");
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 496-499
```cpp
 496:   using LoadKeys = cub::WarpLoad<scalar_t, items_per_thread, cub::WARP_LOAD_TRANSPOSE>;
 497:   using Sort = cub::WarpMergeSort<scalar_t, items_per_thread, WARP_SIZE, int64_t>;
 498:   using StoreKeys = cub::WarpStore<scalar_t, items_per_thread, cub::WARP_STORE_TRANSPOSE>;
 499:   using StoreIndices = cub::WarpStore<int64_t, items_per_thread, cub::WARP_STORE_TRANSPOSE>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 501-506
```cpp
 501:   __shared__ union {
 502:     typename LoadKeys::TempStorage load_keys;
 503:     typename Sort::TempStorage sort;
 504:     typename StoreKeys::TempStorage store_keys;
 505:     typename StoreIndices::TempStorage store_indices;
 506:   } tmp_storage[max_block_dim_y];
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 508-508
```cpp
 508:   auto& warp_storage = tmp_storage[threadIdx.y];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 510-510
```cpp
 510:   // Thread-local arrays for values and indices
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 511-512
```cpp
 511:   scalar_t local_values[items_per_thread];
 512:   int64_t local_indices[items_per_thread];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 514-514
```cpp
 514:   // Invalid sentinel for padding
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 515-517
```cpp
 515:   const scalar_t invalid_value = is_descending
 516:       ? -std::numeric_limits<scalar_t>::infinity()
 517:       : std::numeric_limits<scalar_t>::infinity();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 519-520
```cpp
 519:   // Initialize indices for this slice in blocked arrangement
 520:   // WARP_LOAD_TRANSPOSE uses blocked layout: thread t gets items [t*items_per_thread, t*items_per_thread+items_per_thread-1]
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 521-521
```cpp
 521:   #pragma unroll
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 522-524
```cpp
 522:   for (int i = 0; i < items_per_thread; ++i) {
 523:     local_indices[i] = threadIdx.x * items_per_thread + i;
 524:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 526-526
```cpp
 526:   // Load values from input
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 527-528
```cpp
 527:   LoadKeys(warp_storage.load_keys).Load(input_iter, local_values, inputSliceSize, invalid_value);
 528:   WARP_SYNC();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 530-532
```cpp
 530:   // Sort values with their indices using comparison-based sort
 531:   // For largest=true (descending), use GTOp to get larger values first
 532:   // For largest=false (ascending), use LTOp to get smaller values first
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 533-538
```cpp
 533:   if constexpr (is_descending) {
 534:     Sort(warp_storage.sort).StableSort(local_values, local_indices, GTOp<scalar_t, true>(), inputSliceSize, invalid_value);
 535:   } else {
 536:     Sort(warp_storage.sort).StableSort(local_values, local_indices, LTOp<scalar_t, true>(), inputSliceSize, invalid_value);
 537:   }
 538:   WARP_SYNC();
```
- EN: This block defines or continues the implementation of `constexpr`.
- CN: 该代码块定义或继续实现 `constexpr`。

### Lines 540-540
```cpp
 540:   // Store top-k results
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 541-544
```cpp
 541:   StoreKeys(warp_storage.store_keys).Store(topK_iter, local_values, k);
 542:   WARP_SYNC();
 543:   StoreIndices(warp_storage.store_indices).Store(indices_iter, local_indices, k);
 544: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 546-566
```cpp
 546: template <typename scalar_t, typename IndexType, int Dim>
 547: void launch(
 548:     at::cuda::detail::TensorInfo<const scalar_t, IndexType> input,
 549:     IndexType inputSliceSize,
 550:     IndexType k,
 551:     bool largest,
 552:     IndexType numInputSlices,
 553:     IndexType inputWithinSliceStride,
 554:     at::cuda::detail::TensorInfo<scalar_t, IndexType> topK,
 555:     IndexType topKWithinSliceStride,
 556:     at::cuda::detail::TensorInfo<int64_t, IndexType> indices,
 557:     IndexType indicesWithinSliceStride) {
 558:   TORCH_INTERNAL_ASSERT(inputSliceSize <= MAX_WARP_TOPK_SLICE,
 559:                         "warp topk path requires slice size <= ", MAX_WARP_TOPK_SLICE);
 560: 
 561:   const auto stream = c10::cuda::getCurrentCUDAStream();
 562:   // Use only 1 row per block to minimize shared memory usage
 563:   // (max_block_dim_y * sizeof(union) must fit in 64KB LDS limit)
 564:   constexpr int max_block_dim_y = 1;
 565:   const int block_x = at::cuda::warp_size();
 566:   dim3 block(block_x, max_block_dim_y);
```
- EN: This block defines or continues the implementation of `launch`.
- CN: 该代码块定义或继续实现 `launch`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 568-570
```cpp
 568:   dim3 grid;
 569:   TORCH_INTERNAL_ASSERT(getGridFromTiles(numInputSlices, grid),
 570:                         "Too many slices for warp topk");
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 572-573
```cpp
 572:   // Dispatch based on sort size and sort direction
 573:   // See Note [warp merge sort WARP_SIZE template param]
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 574-575
```cpp
 574: #ifdef USE_ROCM
 575:   #define LAUNCH_KERNEL(SORT_SIZE, IS_DESCENDING) \
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 576-589
```cpp
 576:   if (at::cuda::warp_size() == 32) { \
 577:     warpMergeSortTopK<Dim, Dim, SORT_SIZE, max_block_dim_y, scalar_t, IndexType, IS_DESCENDING, 32> \
 578:       <<<grid, block, 0, stream>>>( \
 579:           input, inputSliceSize, k, numInputSlices, inputWithinSliceStride, \
 580:           topK, topKWithinSliceStride, indices, indicesWithinSliceStride); \
 581:   } \
 582:   else { \
 583:     warpMergeSortTopK<Dim, Dim, SORT_SIZE, max_block_dim_y, scalar_t, IndexType, IS_DESCENDING, 64> \
 584:       <<<grid, block, 0, stream>>>( \
 585:           input, inputSliceSize, k, numInputSlices, inputWithinSliceStride, \
 586:           topK, topKWithinSliceStride, indices, indicesWithinSliceStride); \
 587:   } \
 588:   C10_CUDA_KERNEL_LAUNCH_CHECK()
 589: #else
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 590-590
```cpp
 590:   #define LAUNCH_KERNEL(SORT_SIZE, IS_DESCENDING) \
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 591-596
```cpp
 591:     warpMergeSortTopK<Dim, Dim, SORT_SIZE, max_block_dim_y, scalar_t, IndexType, IS_DESCENDING, 32> \
 592:       <<<grid, block, 0, stream>>>( \
 593:           input, inputSliceSize, k, numInputSlices, inputWithinSliceStride, \
 594:           topK, topKWithinSliceStride, indices, indicesWithinSliceStride); \
 595:     C10_CUDA_KERNEL_LAUNCH_CHECK()
 596: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 598-600
```cpp
 598:   // We have specialized launches for different sizes, as sort_size affects
 599:   // shared memory, registers per thread and occupancy. We can use 'LAUNCH_KERNEL(512, false);'
 600:   // however, that results in lower performance.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 601-622
```cpp
 601:   if (largest) {
 602:     if (inputSliceSize <= 64) {
 603:       LAUNCH_KERNEL(64, true);
 604:     } else if (inputSliceSize <= 128) {
 605:       LAUNCH_KERNEL(128, true);
 606:     } else if (inputSliceSize <= 256) {
 607:       LAUNCH_KERNEL(256, true);
 608:     } else {
 609:       // inputSliceSize <= 512
 610:       LAUNCH_KERNEL(512, true);
 611:     }
 612:   } else {
 613:     if (inputSliceSize <= 64) {
 614:       LAUNCH_KERNEL(64, false);
 615:     } else if (inputSliceSize <= 128) {
 616:       LAUNCH_KERNEL(128, false);
 617:     } else if (inputSliceSize <= 256) {
 618:       LAUNCH_KERNEL(256, false);
 619:     } else {
 620:       // inputSliceSize <= 512
 621:       LAUNCH_KERNEL(512, false);
 622:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 623-623
```cpp
 623:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 625-625
```cpp
 625:   #undef LAUNCH_KERNEL
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 626-626
```cpp
 626: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 628-629
```cpp
 628: } // namespace warptopk
 629: #endif // defined(USE_ROCM) && HAS_WARP_MERGE_SORT()
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 631-652
```cpp
 631: namespace mbtopk { // multi_block_topk
 632: 
 633: // Assumptions:
 634: // The number of elements can be larger than UINT32_MAX, but
 635: // the number of total blocks can not be larger than UINT32_MAX.
 636: // So we can not have more than UINT32_MAX slices. The actual limit
 637: // for number of slices could be a few fold smaller than UINT32_MAX,
 638: // because we could be using multiple blocks per slice.
 639: // Further more, the size of each input slice is also assumped to be
 640: // smaller than UINT32_MAX
 641: 
 642: constexpr int BLOCK_THREADS = 256;
 643: 
 644: // Over what radix we are selecting values
 645: constexpr int RADIX_BITS = 8;
 646: constexpr int RADIX_DIGITS = 1 << RADIX_BITS; // 2 ^ RADIX_BITS
 647: constexpr int RADIX_MASK = (RADIX_DIGITS - 1);
 648: static_assert(RADIX_DIGITS <= BLOCK_THREADS, "RADIX_DIGITS must be <= BLOCK_THREADS");
 649: constexpr int MIN_ITEMS_PER_THREAD = 4;
 650: #if defined(USE_ROCM)
 651: // AMD: Allow higher items_per_thread for large arrays to reduce blocks_per_slice
 652: // This reduces overhead in accumulation loops (computeBlockwiseWithinKCounts, gatherTopK)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 653-656
```cpp
 653: constexpr int MAX_ITEMS_PER_THREAD = 96;
 654: #else
 655: constexpr int MAX_ITEMS_PER_THREAD = 64;
 656: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 658-664
```cpp
 658: template <typename T, typename IndexType>
 659: __global__ void fill(T* x, T value, IndexType size) {
 660:   IndexType idx = blockIdx.x * blockDim.x + threadIdx.x;
 661:   for (IndexType i = idx; i < size; i += gridDim.x * blockDim.x) {
 662:     x[i] = value;
 663:   }
 664: }
```
- EN: This block defines GPU kernel entry point(s) `fill`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `fill`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 666-666
```cpp
 666: // compute local histogram for each block
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 667-680
```cpp
 667: template <typename T, typename IndexType, typename Bitwise, int Dim>
 668: C10_LAUNCH_BOUNDS_1(BLOCK_THREADS)
 669: __global__ void computeBlockDigitCounts(
 670:     at::cuda::detail::TensorInfo<const T, IndexType> input,
 671:     uint32_t slice_size,
 672:     uint32_t* ks_to_find,  // size: num_slices, unused arg but for mysterious reasons perf is better when it's present
 673:     uint32_t num_slices,
 674:     IndexType withinSliceStride,
 675:     int current_bit,
 676:     int items_per_thread,
 677:     uint32_t blocks_per_slice,
 678:     Bitwise desiredMask,
 679:     Bitwise* desires,      // size: num_slices
 680:     short* counts         // size: num_slices * blocks_per_slice * radix_digits
```
- EN: This block defines GPU kernel entry point(s) `computeBlockDigitCounts`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `computeBlockDigitCounts`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 681-701
```cpp
 681:   ) {
 682: 
 683:   int items_per_block = items_per_thread * BLOCK_THREADS;
 684:   int tidx = threadIdx.x;
 685:   uint32_t block_idx = getLinearBlockId<uint32_t>();
 686:   uint32_t slice_idx = block_idx / blocks_per_slice;
 687:   uint32_t blk_idx_in_slice = block_idx % blocks_per_slice;
 688:   if (slice_idx >= num_slices) {
 689:     return;
 690:   }
 691: 
 692:   Bitwise desired = desires[slice_idx];
 693:   IndexType slice_start_index = at::cuda::detail::IndexToOffset<const T, IndexType, Dim>::get(slice_idx, input);
 694:   const T* data = &input.data[slice_start_index];
 695: 
 696:   static_assert(MAX_ITEMS_PER_THREAD * BLOCK_THREADS < std::numeric_limits<short>::max(),
 697:     "blockwise counter too large");
 698:   union __align__(16) TempStorage {
 699:     uint32_t digit_counters[RADIX_DIGITS];
 700:   };
 701:   __shared__ TempStorage temp_storage;
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 703-703
```cpp
 703:   // fill digit_counters with zeros
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 704-707
```cpp
 704:   if (tidx < RADIX_DIGITS) {
 705:     temp_storage.digit_counters[tidx] = 0;
 706:   }
 707:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 709-711
```cpp
 709:   items_per_thread = (blk_idx_in_slice + 1 < blocks_per_slice)
 710:       ? items_per_thread
 711:       : at::ceil_div((int64_t)(slice_size - blk_idx_in_slice * items_per_block), (int64_t)BLOCK_THREADS);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 713-713
```cpp
 713:   // collect digit counts and store in shared memory
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 714-726
```cpp
 714:   for (int i = 0; i < items_per_thread; ++i) {
 715:     // Find the start offset for this slice
 716:     IndexType idx = blk_idx_in_slice * items_per_block + i * BLOCK_THREADS + tidx;
 717:     if (idx < slice_size) {
 718:       idx *= withinSliceStride;
 719:       Bitwise val = TopKTypeConfig<T>::convert(doLdg(&data[idx]));
 720:       bool has_val = ((val & desiredMask) == (desired & desiredMask));
 721:       Bitwise digit = at::cuda::Bitfield<Bitwise>::getBitfield(val, current_bit, RADIX_BITS);
 722:       if (has_val) {
 723:         atomicAdd(&temp_storage.digit_counters[digit], 1);
 724:       }
 725:     }
 726:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 728-728
```cpp
 728:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 730-730
```cpp
 730:   // load digit counter to register, one digit per thread
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 731-735
```cpp
 731:   static_assert(RADIX_DIGITS <= BLOCK_THREADS, "this kernel requires RADIX_DIGITS <= BLOCK_THREADS");
 732:   uint32_t digit_count = 0;
 733:   if (tidx < RADIX_DIGITS) {
 734:     digit_count = temp_storage.digit_counters[tidx];
 735:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 737-738
```cpp
 737:   // We always write out counts regardless if blocks_per_slice == 1 because
 738:   // it will be used to compute offsets for `gatherTopK`.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 739-742
```cpp
 739:   if (tidx < RADIX_DIGITS) {
 740:     counts[block_idx * RADIX_DIGITS + tidx] = digit_count;
 741:   }
 742: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 744-744
```cpp
 744: #ifndef USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 745-745
```cpp
 745: // CUDA path: compute global histogram and cumsum for each row
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 746-767
```cpp
 746: __global__ void computeDigitCumSum(
 747:   short* counts,
 748:   uint32_t* digit_cum_sum,
 749:   uint32_t blocks_per_slice) {
 750:   int tidx = threadIdx.x + blockIdx.x * blockDim.x;
 751:   int digit_idx = threadIdx.x;
 752:   uint32_t slice_idx = blockIdx.x;
 753: 
 754:   typedef cub::BlockScan<uint32_t, RADIX_DIGITS> BlockScan;
 755:   __shared__ typename BlockScan::TempStorage scan_storage;
 756:   // accumulates counters from multiple blocks
 757:   uint32_t digit_count = 0;
 758:   if (threadIdx.x < RADIX_DIGITS) {
 759:     constexpr int HISTO_ACCUM_TILE = 4;
 760:     uint32_t rounds = blocks_per_slice / HISTO_ACCUM_TILE;
 761:     for (int iter = 0; iter < rounds; iter++)  {
 762:       int base = HISTO_ACCUM_TILE * iter;
 763:       #pragma unroll
 764:       for (int j = 0; j < HISTO_ACCUM_TILE; j++) {
 765:         int blk = base + j;
 766:         digit_count += counts[(slice_idx * blocks_per_slice + blk) * RADIX_DIGITS + digit_idx];
 767:       }
```
- EN: This block defines GPU kernel entry point(s) `computeDigitCumSum`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `computeDigitCumSum`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 768-771
```cpp
 768:     }
 769:     for (int blk = HISTO_ACCUM_TILE * rounds; blk < blocks_per_slice; blk++)  {
 770:       digit_count += counts[(slice_idx * blocks_per_slice + blk) * RADIX_DIGITS + digit_idx];
 771:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 773-781
```cpp
 773:   }
 774:   // compute the block-wide inclusive prefix sum
 775:   uint32_t digit_count_cumsum;
 776:   BlockScan(scan_storage).InclusiveSum(digit_count, digit_count_cumsum);
 777:   __syncthreads();
 778:   if (threadIdx.x < RADIX_DIGITS) {
 779:     digit_cum_sum[tidx] = digit_count_cumsum;
 780:   }
 781: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 783-783
```cpp
 783: // CUDA path: Assumption: k can not be larger than UINT32_MAX
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 784-797
```cpp
 784: template <typename Bitwise, typename T>
 785: C10_LAUNCH_BOUNDS_1(RADIX_DIGITS)  // one thread per digit
 786: __global__ void computeBlockwiseWithinKCounts(
 787:   Bitwise* desires_in,          // size: num_slices
 788:   short* counts,                // size: num_slices * blocks_per_slice * radix_digits
 789:   uint32_t* digit_cum_sum,      // CUDA: reads pre-computed cumsum
 790:   uint32_t* ks_to_find_in,      // size: num_slices
 791:   uint32_t blocks_per_slice,
 792:   int current_bit,
 793:   bool largest,
 794:   // outputs:
 795:   uint32_t* withinKCounts,      // size: num_slices * blocks_per_slice == num_blocks
 796:   T* kthValues,                 // size: num_slices, only write when current_bit reaches 0
 797:   uint32_t* ks_to_find_out,
```
- EN: This block defines GPU kernel entry point(s) `computeBlockwiseWithinKCounts`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `computeBlockwiseWithinKCounts`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 798-819
```cpp
 798:   Bitwise* desires_out,
 799:   uint32_t num_blocks
 800: ) {
 801:   // This kernel should be launched with the same number of blocks as the `computeBlockDigitCounts` kernel.
 802:   int tidx = threadIdx.x;
 803:   uint32_t block_idx = getLinearBlockId<uint32_t>();
 804:   uint32_t slice_idx = block_idx / blocks_per_slice;
 805: 
 806:   // The grid is computed from `getGridFromTiles`, when there are lots of
 807:   // elements, we will use both blockIdx.x and blockIdx.y, and maybe blockIdx.z
 808:   // when this is the case, the number of blocks that we are launching can be
 809:   // more than the number of blocks we need. So we need to check the range of
 810:   // `block_idx`.
 811:   if (block_idx >= num_blocks) {
 812:     return;
 813:   }
 814: 
 815: 
 816:   __shared__ Bitwise desired;
 817:   uint32_t k_to_find = ks_to_find_in[slice_idx];
 818: 
 819:   if (tidx < RADIX_DIGITS) {
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 820-822
```cpp
 820:     uint32_t position = slice_idx * RADIX_DIGITS + tidx;
 821:     uint32_t digit_count_cumsum = digit_cum_sum[position];
 822:     uint32_t digit_count_cumsum_left = (tidx == 0) ? 0 : digit_cum_sum[position - 1];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 824-826
```cpp
 824:     // if not the last pass: update desired and ks_to_find
 825:     // if last pass: write out the kth value
 826:     // only one thread in block enters this condition
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 827-840
```cpp
 827:     if (digit_count_cumsum_left < k_to_find && k_to_find <= digit_count_cumsum) {
 828:       desired = desires_in[slice_idx];
 829:       desired = at::cuda::Bitfield<Bitwise>::setBitfield(desired, tidx, current_bit, RADIX_BITS);
 830:       // let a single block per slice update the values
 831:       if (block_idx == slice_idx * blocks_per_slice) {
 832:         desires_out[slice_idx] = desired;
 833:         if (current_bit > 0) {
 834:           ks_to_find_out[slice_idx] = k_to_find - digit_count_cumsum_left;
 835:         } else {
 836:           kthValues[slice_idx] = TopKTypeConfig<T>::deconvert(desired);
 837:         }
 838:       }
 839:     }
 840:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 841-841
```cpp
 841:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 843-843
```cpp
 843:   Bitwise desired_digit = at::cuda::Bitfield<Bitwise>::getBitfield(desired, current_bit, RADIX_BITS);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 845-849
```cpp
 845:   // if largest, then only threads that has tidx > desired_digit are active
 846:   // if !largest, then only threads that has tidx < desired_digit are active
 847:   // each active thread will read the count for its corresponding, and
 848:   // do warp reduction followed by shared memory reduction to get the total count
 849:   // non-active thread should not load, and non-active warp should not do reduction.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 850-869
```cpp
 850:   bool warp_is_active, thread_is_active;
 851:   int warp = tidx / C10_WARP_SIZE;
 852:   if (largest) {
 853:     int end_of_warp = warp * C10_WARP_SIZE + C10_WARP_SIZE - 1;
 854:     warp_is_active = end_of_warp > desired_digit;
 855:     thread_is_active = tidx > desired_digit;
 856:   } else {
 857:     int start_of_warp = warp * C10_WARP_SIZE;
 858:     warp_is_active = start_of_warp < desired_digit;
 859:     thread_is_active = tidx < desired_digit;
 860:   }
 861:   uint32_t count = 0;
 862:   if (warp_is_active) {
 863:     if (thread_is_active) {
 864:       count = doLdg(counts + block_idx * RADIX_DIGITS + tidx);
 865:     }
 866:     for (int offset = C10_WARP_SIZE / 2; offset > 0; offset /= 2) {
 867:       count += WARP_SHFL_DOWN(count, offset);
 868:     }
 869:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 871-886
```cpp
 871:   constexpr int SHMEM_SIZE = RADIX_DIGITS / C10_WARP_SIZE_LOWER_BOUND;  // max shmem size on ROCm
 872:   const int num_warps = RADIX_DIGITS / C10_WARP_SIZE;
 873:   __shared__ uint32_t warp_counts[SHMEM_SIZE];
 874:   if (tidx % C10_WARP_SIZE == 0) {
 875:     warp_counts[warp] = count;
 876:   }
 877:   __syncthreads();
 878:   static_assert(RADIX_DIGITS < C10_WARP_SIZE * C10_WARP_SIZE,
 879:     "Assuming only 1 warp is needed for final reduction");
 880:   if (warp != 0) {
 881:     return;
 882:   }
 883:   count = 0;
 884:   if (tidx < num_warps) {
 885:     count = warp_counts[tidx];
 886:   }
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 887-893
```cpp
 887:   for (int offset = num_warps / 2; offset > 0; offset /= 2) {
 888:     count += WARP_SHFL_DOWN(count, offset);
 889:   }
 890:   if (tidx == 0) {
 891:     withinKCounts[block_idx] += count;
 892:   }
 893: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 895-895
```cpp
 895: // CUDA path: Assumption: slice_size can not be larger than UINT32_MAX
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 896-911
```cpp
 896: template <typename Bitwise>
 897: __global__ void computeBlockwiseKthCounts(
 898:   Bitwise* desires,            // size: num_slices
 899:   short* counts,               // size: num_slices * blocks_per_slice * radix_digits
 900:   uint32_t num_blocks,         // the number of blocks used by `computeBlockDigitCounts` kernel
 901:   uint32_t blocks_per_slice,
 902:   // outputs:
 903:   uint32_t* kthCounts          // size: num_slices * blocks_per_slice == num_blocks
 904: ) {
 905:   CUDA_KERNEL_LOOP_TYPE(idx, num_blocks, uint32_t) {
 906:     uint32_t slice_idx = idx / blocks_per_slice;
 907:     Bitwise desired = doLdg(desires + slice_idx);
 908:     Bitwise desired_digit = at::cuda::Bitfield<Bitwise>::getBitfield(desired, 0, RADIX_BITS);
 909:     kthCounts[idx] = doLdg(counts + idx * RADIX_DIGITS + desired_digit);
 910:   }
 911: }
```
- EN: This block defines GPU kernel entry point(s) `computeBlockwiseKthCounts`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `computeBlockwiseKthCounts`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 913-913
```cpp
 913: #else // USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 915-915
```cpp
 915: // ROCm path: Assumption: k can not be larger than UINT32_MAX
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 916-929
```cpp
 916: template <typename Bitwise, typename T>
 917: C10_LAUNCH_BOUNDS_1(RADIX_DIGITS)  // one thread per digit
 918: __global__ void computeBlockwiseWithinKCounts(
 919:   Bitwise* desires_in,          // size: num_slices
 920:   short* counts,                // size: num_slices * blocks_per_slice * radix_digits
 921:   uint32_t* ks_to_find_in,      // size: num_slices
 922:   uint32_t blocks_per_slice,
 923:   int current_bit,
 924:   bool largest,
 925:   // outputs:
 926:   uint32_t* withinKCounts,      // size: num_slices * blocks_per_slice == num_blocks
 927:   T* kthValues,                 // size: num_slices, only write when current_bit reaches 0
 928:   uint32_t* ks_to_find_out,
 929:   Bitwise* desires_out,
```
- EN: This block defines GPU kernel entry point(s) `computeBlockwiseWithinKCounts`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `computeBlockwiseWithinKCounts`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 930-951
```cpp
 930:   uint32_t* kthCounts,          // ROCm: added kthCounts output
 931:   uint32_t num_blocks
 932: ) {
 933:   // This kernel should be launched with the same number of blocks as the `computeBlockDigitCounts` kernel.
 934:   int tidx = threadIdx.x;
 935:   uint32_t block_idx = getLinearBlockId<uint32_t>();
 936:   uint32_t slice_idx = block_idx / blocks_per_slice;
 937: 
 938:   // The grid is computed from `getGridFromTiles`, when there are lots of
 939:   // elements, we will use both blockIdx.x and blockIdx.y, and maybe blockIdx.z
 940:   // when this is the case, the number of blocks that we are launching can be
 941:   // more than the number of blocks we need. So we need to check the range of
 942:   // `block_idx`.
 943:   if (block_idx >= num_blocks) {
 944:     return;
 945:   }
 946: 
 947: 
 948:   __shared__ Bitwise desired;
 949:   uint32_t k_to_find = ks_to_find_in[slice_idx];
 950: 
 951:   // Use hipCUB BlockScan for efficient inclusive scan
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 952-953
```cpp
 952:   typedef ROCM_HIPCUB(at_cuda_detail::cub)::BlockScan<uint32_t, RADIX_DIGITS> BlockScan;
 953:   __shared__ typename BlockScan::TempStorage scan_storage;
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 955-955
```cpp
 955:   // Build per-slice digit totals in shared memory and compute inclusive cumsum
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 956-977
```cpp
 956:   __shared__ uint32_t digit_totals[RADIX_DIGITS];
 957:   if (tidx < RADIX_DIGITS) {
 958:     uint32_t sum = 0;
 959:     // Accumulate counts across all blocks in the slice for this digit
 960: 
 961:     // AMD optimization: Improve memory access pattern to reduce latency
 962:     // Access pattern: counts[base + blk * RADIX_DIGITS + tidx]
 963:     // For large blocks_per_slice, this loop dominates kernel time
 964:     // Unroll by 4 to improve instruction-level parallelism and hide latency
 965:     const short* count_ptr = counts + slice_idx * blocks_per_slice * RADIX_DIGITS + tidx;
 966:     uint32_t blk = 0;
 967:     // Process 4 blocks at a time to improve ILP
 968:     for (; blk + 3 < blocks_per_slice; blk += 4) {
 969:       uint32_t v0 = count_ptr[0 * RADIX_DIGITS];
 970:       uint32_t v1 = count_ptr[1 * RADIX_DIGITS];
 971:       uint32_t v2 = count_ptr[2 * RADIX_DIGITS];
 972:       uint32_t v3 = count_ptr[3 * RADIX_DIGITS];
 973:       sum += v0 + v1 + v2 + v3;
 974:       count_ptr += 4 * RADIX_DIGITS;
 975:     }
 976:     // Handle remaining blocks
 977:     for (; blk < blocks_per_slice; ++blk) {
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 978-983
```cpp
 978:       sum += count_ptr[0];
 979:       count_ptr += RADIX_DIGITS;
 980:     }
 981:     digit_totals[tidx] = sum;
 982:   }
 983:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 985-985
```cpp
 985:   // Use hipCUB BlockScan for efficient inclusive scan (replaces manual scan)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 986-993
```cpp
 986:   uint32_t digit_total = (tidx < RADIX_DIGITS) ? digit_totals[tidx] : 0u;
 987:   uint32_t digit_count_cumsum;
 988:   BlockScan(scan_storage).InclusiveSum(digit_total, digit_count_cumsum);
 989:   __syncthreads();
 990:   if (tidx < RADIX_DIGITS) {
 991:     digit_totals[tidx] = digit_count_cumsum;
 992:   }
 993:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 995-1015
```cpp
 995:   if (tidx < RADIX_DIGITS) {
 996:     // digit_count_cumsum already contains the correct value from BlockScan
 997:     uint32_t digit_count_cumsum_left = (tidx == 0) ? 0 : digit_totals[tidx - 1];
 998: 
 999:     // if not the last pass: update desired and ks_to_find
1000:     // if last pass: write out the kth value
1001:     // only one thread in block enters this condition
1002:     if (digit_count_cumsum_left < k_to_find && k_to_find <= digit_count_cumsum) {
1003:       desired = desires_in[slice_idx];
1004:       desired = at::cuda::Bitfield<Bitwise>::setBitfield(desired, tidx, current_bit, RADIX_BITS);
1005:       // let a single block per slice update the values
1006:       if (block_idx == slice_idx * blocks_per_slice) {
1007:         desires_out[slice_idx] = desired;
1008:         if (current_bit > 0) {
1009:           ks_to_find_out[slice_idx] = k_to_find - digit_count_cumsum_left;
1010:         } else {
1011:           kthValues[slice_idx] = TopKTypeConfig<T>::deconvert(desired);
1012:         }
1013:       }
1014:     }
1015:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1016-1016
```cpp
1016:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1018-1018
```cpp
1018:   Bitwise desired_digit = at::cuda::Bitfield<Bitwise>::getBitfield(desired, current_bit, RADIX_BITS);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1020-1024
```cpp
1020:   // if largest, then only threads that has tidx > desired_digit are active
1021:   // if !largest, then only threads that has tidx < desired_digit are active
1022:   // each active thread will read the count for its corresponding, and
1023:   // do warp reduction followed by shared memory reduction to get the total count
1024:   // non-active thread should not load, and non-active warp should not do reduction.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 1025-1044
```cpp
1025:   bool warp_is_active, thread_is_active;
1026:   int warp = tidx / C10_WARP_SIZE;
1027:   if (largest) {
1028:     int end_of_warp = warp * C10_WARP_SIZE + C10_WARP_SIZE - 1;
1029:     warp_is_active = end_of_warp > desired_digit;
1030:     thread_is_active = tidx > desired_digit;
1031:   } else {
1032:     int start_of_warp = warp * C10_WARP_SIZE;
1033:     warp_is_active = start_of_warp < desired_digit;
1034:     thread_is_active = tidx < desired_digit;
1035:   }
1036:   uint32_t count = 0;
1037:   if (warp_is_active) {
1038:     if (thread_is_active) {
1039:       count = doLdg(counts + block_idx * RADIX_DIGITS + tidx);
1040:     }
1041:     for (int offset = C10_WARP_SIZE / 2; offset > 0; offset /= 2) {
1042:       count += WARP_SHFL_DOWN(count, offset);
1043:     }
1044:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1046-1052
```cpp
1046:   constexpr int SHMEM_SIZE = RADIX_DIGITS / C10_WARP_SIZE_LOWER_BOUND; // max shmem size on ROCm
1047:   const int num_warps = RADIX_DIGITS / C10_WARP_SIZE;
1048:   __shared__ uint32_t warp_counts[SHMEM_SIZE];
1049:   if (tidx % C10_WARP_SIZE == 0) {
1050:     warp_counts[warp] = count;
1051:   }
1052:   __syncthreads();
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 1054-1067
```cpp
1054:   CUDA_KERNEL_ASSERT(RADIX_DIGITS < C10_WARP_SIZE * C10_WARP_SIZE);
1055:   if (warp != 0) {
1056:     return;
1057:   }
1058:   count = 0;
1059:   if (tidx < num_warps) {
1060:     count = warp_counts[tidx];
1061:   }
1062:   for (int offset = num_warps / 2; offset > 0; offset /= 2) {
1063:     count += WARP_SHFL_DOWN(count, offset);
1064:   }
1065:   if (tidx == 0) {
1066:     withinKCounts[block_idx] += count;
1067:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1069-1069
```cpp
1069:   // On the last pass (current_bit == 0), write out block-wise kthCounts directly
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1070-1074
```cpp
1070:   if (tidx == 0 && current_bit == 0) {
1071:     Bitwise desired_digit0 = at::cuda::Bitfield<Bitwise>::getBitfield(desired, 0, RADIX_BITS);
1072:     kthCounts[block_idx] = doLdg(counts + block_idx * RADIX_DIGITS + desired_digit0);
1073:   }
1074: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1076-1076
```cpp
1076: #endif // USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1078-1083
```cpp
1078: template <typename T, typename IndexType, int Dim>
1079: C10_LAUNCH_BOUNDS_1(BLOCK_THREADS)
1080: __global__ void gatherTopK(at::cuda::detail::TensorInfo<const T, IndexType> input,
1081:                            IndexType inputSliceSize,
1082:                            IndexType outputSliceSize, // aka `k`
1083:                            bool largest,
```
- EN: This block defines GPU kernel entry point(s) `gatherTopK`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `gatherTopK`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 1085-1086
```cpp
1085:                            uint32_t numInputSlices,
1086:                            IndexType inputWithinSliceStride,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1088-1089
```cpp
1088:                            at::cuda::detail::TensorInfo<T, IndexType> topK,
1089:                            IndexType topKWithinSliceStride,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1091-1092
```cpp
1091:                            at::cuda::detail::TensorInfo<int64_t, IndexType> indices,
1092:                            IndexType indicesWithinSliceStride,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1094-1095
```cpp
1094:                            uint32_t items_per_thread,
1095:                            uint32_t blocks_per_slice,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1097-1118
```cpp
1097:                            T *kthValues,
1098:                            uint32_t* withinKCounts,
1099:                            uint32_t* kthCounts,
1100:                            uint32_t num_blocks) {
1101: 
1102:   uint32_t items_per_block = items_per_thread * BLOCK_THREADS;
1103:   uint32_t tidx = threadIdx.x;
1104:   uint32_t block_idx = getLinearBlockId<uint32_t>();
1105: 
1106:   // The grid is computed from `getGridFromTiles`, when there are lots of
1107:   // elements, we will use both blockIdx.x and blockIdx.y, and maybe blockIdx.z
1108:   // when this is the case, the number of blocks that we are launching can be
1109:   // more than the number of blocks we need. So we need to check the range of
1110:   // `block_idx`.
1111:   if (block_idx >= num_blocks) {
1112:     return;
1113:   }
1114: 
1115:   uint32_t slice_idx = block_idx / blocks_per_slice;
1116:   uint32_t blk_idx_in_slice = block_idx % blocks_per_slice;
1117: 
1118:   items_per_thread = (blk_idx_in_slice + 1 < blocks_per_slice)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1119-1120
```cpp
1119:       ? items_per_thread
1120:       : at::ceil_div((int64_t)(inputSliceSize - blk_idx_in_slice * items_per_block), (int64_t)BLOCK_THREADS);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1122-1122
```cpp
1122:   // Find the start offset for our slice
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1123-1128
```cpp
1123:   IndexType sliceStartIndex =
1124:     at::cuda::detail::IndexToOffset<const T, IndexType, Dim>::get(slice_idx, input);
1125:   IndexType topKSliceStartIndex =
1126:     at::cuda::detail::IndexToOffset<T, IndexType, Dim>::get(slice_idx, topK);
1127:   IndexType indicesSliceStartIndex =
1128:     at::cuda::detail::IndexToOffset<int64_t, IndexType, Dim>::get(slice_idx, indices);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1130-1132
```cpp
1130:   const T* inputSliceStart = &input.data[sliceStartIndex];
1131:   T* topKSliceStart = &topK.data[topKSliceStartIndex];
1132:   int64_t* indicesSliceStart = &indices.data[indicesSliceStartIndex];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1134-1134
```cpp
1134:   // Find the k-th highest element in our input
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1135-1136
```cpp
1135:   T kthValue = kthValues[slice_idx];
1136:   const auto kthValueConverted = at::native::TopKTypeConfig<T>::convert(kthValue);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1138-1138
```cpp
1138:   // Find the start index in output tensor of this block
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1139-1139
```cpp
1139: #if !defined(USE_ROCM)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1140-1140
```cpp
1140:   // CUDA path: Use pre-computed prefix sums from CUB
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1141-1162
```cpp
1141:   uint32_t startWithinK = 0;
1142:   if (blk_idx_in_slice > 0) {
1143:     startWithinK = withinKCounts[block_idx - 1];
1144:   }
1145:   uint32_t startKth = withinKCounts[slice_idx * blocks_per_slice + blocks_per_slice - 1];
1146:   if (blk_idx_in_slice > 0) {
1147:     startKth += kthCounts[block_idx - 1];
1148:   }
1149: #else
1150:   // ROCm path: Compute prefix sums inline with unrolled loop
1151:   __shared__ uint32_t slice_prefix_within;
1152:   __shared__ uint32_t slice_prefix_kth;
1153:   __shared__ uint32_t slice_total_within;
1154:   if (threadIdx.x == 0) {
1155:     uint32_t prefix_within = 0;
1156:     uint32_t prefix_kth = 0;
1157:     uint32_t total_within = 0;
1158:     uint32_t slice_offset = slice_idx * blocks_per_slice;
1159:     // AMD optimization: Unroll prefix sum loop to improve ILP
1160:     // This loop iterates up to 245 times for 1M case, causing memory latency
1161:     const uint32_t* within_ptr = withinKCounts + slice_offset;
1162:     const uint32_t* kth_ptr = kthCounts + slice_offset;
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 1163-1184
```cpp
1163:     uint32_t blk = 0;
1164:     // Process 4 blocks at a time
1165:     for (; blk + 3 < blocks_per_slice; blk += 4) {
1166:       uint32_t w0 = within_ptr[0];
1167:       uint32_t w1 = within_ptr[1];
1168:       uint32_t w2 = within_ptr[2];
1169:       uint32_t w3 = within_ptr[3];
1170:       total_within += w0 + w1 + w2 + w3;
1171:       if (blk < blk_idx_in_slice) {
1172:         prefix_within += w0;
1173:         prefix_kth += kth_ptr[0];
1174:       }
1175:       if (blk + 1 < blk_idx_in_slice) {
1176:         prefix_within += w1;
1177:         prefix_kth += kth_ptr[1];
1178:       }
1179:       if (blk + 2 < blk_idx_in_slice) {
1180:         prefix_within += w2;
1181:         prefix_kth += kth_ptr[2];
1182:       }
1183:       if (blk + 3 < blk_idx_in_slice) {
1184:         prefix_within += w3;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1185-1198
```cpp
1185:         prefix_kth += kth_ptr[3];
1186:       }
1187:       within_ptr += 4;
1188:       kth_ptr += 4;
1189:     }
1190:     // Handle remaining blocks
1191:     for (; blk < blocks_per_slice; ++blk) {
1192:       uint32_t within_val = *within_ptr++;
1193:       total_within += within_val;
1194:       if (blk < blk_idx_in_slice) {
1195:         prefix_within += within_val;
1196:         prefix_kth += *kth_ptr;
1197:       }
1198:       kth_ptr++;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1199-1204
```cpp
1199:     }
1200:     slice_prefix_within = prefix_within;
1201:     slice_prefix_kth = prefix_kth;
1202:     slice_total_within = total_within;
1203:   }
1204:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1206-1208
```cpp
1206:   uint32_t startWithinK = slice_prefix_within;
1207:   uint32_t startKth = slice_total_within + slice_prefix_kth;
1208: #endif // USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1210-1210
```cpp
1210:   // Read input, select topk out and write
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1211-1232
```cpp
1211:   typedef cub::BlockScan<uint32_t, BLOCK_THREADS> BlockScan;
1212:   __shared__ typename BlockScan::TempStorage temp_storage;
1213:   for (int i = 0; i < items_per_thread; ++i) {
1214:     // Find the start offset for this slice
1215:     IndexType idx = blk_idx_in_slice * items_per_block + i * BLOCK_THREADS + tidx;
1216:     T val;
1217:     int withinK = 0;
1218:     int kth = 0;
1219:     if (idx < inputSliceSize) {
1220:       val = doLdg(inputSliceStart + idx * inputWithinSliceStride);
1221:       const auto valConverted = at::native::TopKTypeConfig<T>::convert(val);
1222:       withinK = (largest ? valConverted > kthValueConverted : valConverted < kthValueConverted);
1223:       kth = (valConverted == kthValueConverted);
1224:     }
1225: 
1226:     uint32_t withinKIndex;
1227:     uint32_t numWithinK;
1228:     BlockScan(temp_storage).ExclusiveSum(withinK, withinKIndex, numWithinK);
1229:     __syncthreads();
1230:     if (withinK) {
1231:       uint32_t offset = withinKIndex + startWithinK;
1232:       topKSliceStart[offset * topKWithinSliceStride] = val;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 1233-1235
```cpp
1233:       indicesSliceStart[offset * indicesWithinSliceStride] = idx;
1234:     }
1235:     startWithinK += numWithinK;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1237-1250
```cpp
1237:     if (startKth < outputSliceSize) {
1238:       uint32_t kthIndex;
1239:       uint32_t numKth;
1240:       BlockScan(temp_storage).ExclusiveSum(kth, kthIndex, numKth);
1241:       __syncthreads();
1242:       if (kth) {
1243:         uint32_t offset = kthIndex + startKth;
1244:         if (offset < outputSliceSize) {
1245:           topKSliceStart[offset * topKWithinSliceStride] = val;
1246:           indicesSliceStart[offset * indicesWithinSliceStride] = idx;
1247:         }
1248:       }
1249:       startKth += numKth;
1250:     }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1251-1252
```cpp
1251:   }
1252: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1254-1275
```cpp
1254: int get_items_per_thread(uint64_t num_slices, uint64_t slice_size) {
1255:   // Occupancy of this kernel is limited by registers per thread
1256:   // Platform-specific tuning for optimal register pressure
1257: #if defined(USE_ROCM)
1258:   // AMD RDNA/CDNA architecture: measured via rocprof for mbtopk kernels
1259:   // MI250X has different register file organization than NVIDIA
1260:   // - More VGPRs available per thread (256 vs 255 on NVIDIA)
1261:   // - Wave64 execution model requires different occupancy tuning
1262:   // Empirically tuned for large 1D TopK (1M elements, k=8 case)
1263:   constexpr int REGS_PER_THREAD = 48;  // Higher register usage acceptable on AMD
1264: #else
1265:   constexpr int REGS_PER_THREAD = 40;  // from nsight launch statistics (NVIDIA)
1266: #endif
1267:   constexpr int REGS_PER_BLOCK = REGS_PER_THREAD * BLOCK_THREADS;
1268: 
1269:   cudaDeviceProp* prop = at::cuda::getCurrentDeviceProperties();
1270:   int mpc = prop->multiProcessorCount;
1271:   int regs_per_mp = prop->regsPerMultiprocessor;
1272:   int max_blocks_per_mp = prop->maxBlocksPerMultiProcessor;
1273:   int blocks_per_mp = std::min(regs_per_mp / REGS_PER_BLOCK, max_blocks_per_mp);
1274: 
1275:   // Calculate items_per_thread to maximize GPU utilization
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `get_items_per_thread`.
- CN: 该代码块定义或继续实现 `get_items_per_thread`。

### Lines 1276-1276
```cpp
1276:   int64_t items_per_thread = at::ceil_div((int64_t)(slice_size * num_slices), (int64_t)(mpc * blocks_per_mp * BLOCK_THREADS));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1278-1278
```cpp
1278: #if defined(USE_ROCM)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1279-1283
```cpp
1279:   // AMD-specific optimization: For large 1D slices, use higher items_per_thread
1280:   // to significantly reduce blocks_per_slice, which reduces overhead in:
1281:   // - computeBlockwiseWithinKCounts accumulation loop (lines 730-750)
1282:   // - gatherTopK prefix sum loop (lines 940-981)
1283:   // Goal: Keep blocks_per_slice under 100 for optimal performance
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1284-1297
```cpp
1284:   if (num_slices <= 4 && slice_size >= 800000) {
1285:     // Very large arrays (800k+): Aggressively increase items_per_thread
1286:     // For 1M elements: items_per_thread=32 → blocks_per_slice=123
1287:     // For 1M elements: items_per_thread=48 → blocks_per_slice=82
1288:     // For 1M elements: items_per_thread=64 → blocks_per_slice=62
1289:     items_per_thread = std::max(items_per_thread, (int64_t)48);
1290:   } else if (num_slices <= 4 && slice_size >= 500000) {
1291:     // Large arrays (500k-800k): Moderately increase items_per_thread
1292:     items_per_thread = std::max(items_per_thread, (int64_t)32);
1293:   } else if (num_slices <= 4 && slice_size >= 250000) {
1294:     // Medium-large arrays (250k-500k): Slightly increase items_per_thread
1295:     items_per_thread = std::max(items_per_thread, (int64_t)24);
1296:   }
1297: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1299-1299
```cpp
1299:   // Clamp to valid range [MIN, MAX] (AMD: [4, 96], CUDA: [4, 64])
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1300-1302
```cpp
1300:   items_per_thread = std::max(MIN_ITEMS_PER_THREAD, std::min((int)items_per_thread, MAX_ITEMS_PER_THREAD));
1301:   return items_per_thread;
1302: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1304-1311
```cpp
1304: class BlockIdxToKey {
1305:   uint32_t blocks_per_slice;
1306: public:
1307:   BlockIdxToKey(uint32_t blocks_per_slice): blocks_per_slice(blocks_per_slice) {}
1308:   __device__ __forceinline__ uint32_t operator()(uint32_t blk) const {
1309:     return blk / blocks_per_slice;
1310:   }
1311: };
```
- EN: This block defines or continues the implementation of `BlockIdxToKey`.
- CN: 该代码块定义或继续实现 `BlockIdxToKey`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 1313-1318
```cpp
1313: template <typename T, typename IndexType, int Dim>
1314: void launch(
1315:     at::cuda::detail::TensorInfo<const T, IndexType> input,
1316:     IndexType inputSliceSize,
1317:     IndexType outputSliceSize, // aka `k`
1318:     bool largest,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1320-1321
```cpp
1320:     uint32_t numInputSlices,
1321:     IndexType inputWithinSliceStride,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1323-1324
```cpp
1323:     at::cuda::detail::TensorInfo<T, IndexType> topK,
1324:     IndexType topKWithinSliceStride,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1326-1347
```cpp
1326:     at::cuda::detail::TensorInfo<int64_t, IndexType> indices,
1327:     IndexType indicesWithinSliceStride) {
1328:   auto stream = c10::cuda::getCurrentCUDAStream();
1329: 
1330:   // configure items_per_thread based on device architecture and input size
1331:   int items_per_thread = get_items_per_thread(numInputSlices, inputSliceSize);
1332:   int items_per_block = items_per_thread * BLOCK_THREADS;
1333: 
1334:   using Bitwise = typename TopKTypeConfig<T>::RadixType;
1335:   uint32_t blocks_per_slice = at::ceil_div((int64_t)inputSliceSize, (int64_t)items_per_block);
1336:   uint32_t num_blocks = numInputSlices * blocks_per_slice;
1337: 
1338:   // temporary storage
1339:   auto& allocator = *c10::cuda::CUDACachingAllocator::get();
1340: 
1341:   auto kthValues_buffer = allocator.allocate(numInputSlices * sizeof(T));
1342:   T* kthValues = reinterpret_cast<T*>(kthValues_buffer.get());
1343: 
1344:   TORCH_CHECK(blocks_per_slice <= std::numeric_limits<uint32_t>::max(), "blocks_per_slice larger than uint32 maximum is not supported");
1345: 
1346: 
1347:   auto ks_to_find_buffer = allocator.allocate(2 * numInputSlices * sizeof(uint32_t));
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1348-1352
```cpp
1348:   uint32_t* ks_to_find = reinterpret_cast<uint32_t*>(ks_to_find_buffer.get());
1349:   uint32_t k_to_find = largest ? inputSliceSize - outputSliceSize + 1: outputSliceSize;
1350:   fill<uint32_t><<<std::min(((int64_t)numInputSlices + 511) / 512, (int64_t)1073741824), 512, 0, stream>>>(
1351:     ks_to_find, k_to_find, numInputSlices);
1352:   C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1354-1355
```cpp
1354:   auto desired_buffer = allocator.allocate(2 * numInputSlices * sizeof(Bitwise));
1355:   Bitwise* desired = reinterpret_cast<Bitwise*>(desired_buffer.get());
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1357-1360
```cpp
1357:   auto counts_buffer = allocator.allocate(num_blocks * RADIX_DIGITS * sizeof(short));
1358:   short* counts = reinterpret_cast<short*>(counts_buffer.get());
1359:   static_assert(MAX_ITEMS_PER_THREAD * BLOCK_THREADS < std::numeric_limits<short>::max(),
1360:     "blockwise counter too large");
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1362-1362
```cpp
1362: #if !defined(USE_ROCM)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1363-1363
```cpp
1363:   // CUDA path: Allocate digit_cum_sum buffer
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1364-1369
```cpp
1364:   auto digit_cum_sum_buffer = allocator.allocate(numInputSlices * RADIX_DIGITS * sizeof(uint32_t));
1365:   uint32_t* digit_cum_sum = reinterpret_cast<uint32_t*>(digit_cum_sum_buffer.get());
1366:   AT_CUDA_CHECK(cudaMemsetAsync(digit_cum_sum, 0, numInputSlices * RADIX_DIGITS * sizeof(uint32_t), stream));
1367: #else
1368:   // ROCm path: No separate digit cumsum buffer; fused into computeBlockwiseWithinKCounts
1369: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1371-1373
```cpp
1371:   auto withinKCounts_buffer = allocator.allocate(num_blocks * sizeof(uint32_t));
1372:   uint32_t* withinKCounts = reinterpret_cast<uint32_t*>(withinKCounts_buffer.get());
1373:   AT_CUDA_CHECK(cudaMemsetAsync(withinKCounts, 0, num_blocks * sizeof(uint32_t), stream));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1375-1376
```cpp
1375:   auto kthCounts_buffer = allocator.allocate(num_blocks * sizeof(uint32_t));
1376:   uint32_t* kthCounts = reinterpret_cast<uint32_t*>(kthCounts_buffer.get());
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1378-1381
```cpp
1378:   Bitwise desiredMask = 0;
1379:   dim3 grid;
1380:   TORCH_INTERNAL_ASSERT(getGridFromTiles(num_blocks, grid), "Too many slices for topk");
1381:   dim3 block(BLOCK_THREADS);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1383-1386
```cpp
1383:   uint32_t * ks_to_find_in = ks_to_find;
1384:   uint32_t * ks_to_find_out = ks_to_find + numInputSlices;
1385:   Bitwise * desired_in = desired;
1386:   Bitwise * desired_out = desired + numInputSlices;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1388-1388
```cpp
1388:   // iterate radix bits for multiple passes
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1389-1410
```cpp
1389:   for (int current_bit = sizeof(T) * 8 - RADIX_BITS; current_bit >= 0; current_bit -= RADIX_BITS) {
1390:     computeBlockDigitCounts<T, IndexType, Bitwise, Dim><<<grid, block, 0, stream>>>(
1391:         input,
1392:         inputSliceSize,
1393:         ks_to_find_in, // unused arg
1394:         numInputSlices,
1395:         inputWithinSliceStride,
1396:         current_bit,
1397:         items_per_thread,
1398:         blocks_per_slice,
1399:         desiredMask,
1400:         desired_in,
1401:         counts);
1402:     C10_CUDA_KERNEL_LAUNCH_CHECK();
1403: 
1404: #if !defined(USE_ROCM)
1405:     // CUDA path: Separate kernel for digit cumsum
1406:     computeDigitCumSum<<<numInputSlices, RADIX_DIGITS, 0, stream>>>(counts, digit_cum_sum, blocks_per_slice);
1407:     C10_CUDA_KERNEL_LAUNCH_CHECK();
1408: 
1409:     // CUDA path: Call computeBlockwiseWithinKCounts with digit_cum_sum, without kthCounts
1410:     computeBlockwiseWithinKCounts<Bitwise, T><<<grid, RADIX_DIGITS, 0, stream>>>(
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1411-1424
```cpp
1411:       desired_in, counts, digit_cum_sum, ks_to_find_in, blocks_per_slice, current_bit, largest,
1412:       withinKCounts, kthValues, ks_to_find_out, desired_out, num_blocks);
1413:     C10_CUDA_KERNEL_LAUNCH_CHECK();
1414: #else
1415:     // ROCm path: Fused version (no digit_cum_sum kernel, includes kthCounts)
1416:     // we unconditionally call this kernel to update desired/ks_to_find/kthValues
1417:     // if cub supports scan_by_key we additionally do k counts
1418:     computeBlockwiseWithinKCounts<Bitwise, T><<<grid, RADIX_DIGITS, 0, stream>>>(
1419:       desired_in, counts, ks_to_find_in, blocks_per_slice, current_bit, largest,
1420:       withinKCounts, kthValues, ks_to_find_out, desired_out, kthCounts, num_blocks);
1421:     C10_CUDA_KERNEL_LAUNCH_CHECK();
1422: #endif
1423:     // swap desired/ks_to_find in and out for next iter
1424:     auto tmp_desired = desired_in;
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1425-1432
```cpp
1425:     desired_in = desired_out;
1426:     desired_out = tmp_desired;
1427:     auto tmp_ks = ks_to_find_in;
1428:     ks_to_find_in = ks_to_find_out;
1429:     ks_to_find_out = tmp_ks;
1430:     desiredMask = at::cuda::Bitfield<Bitwise>::setBitfield(desiredMask, RADIX_MASK, current_bit, RADIX_BITS);
1431:   }
1432:   desired = desired_in;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1434-1434
```cpp
1434: #if !defined(USE_ROCM)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1435-1435
```cpp
1435:   // CUDA path: Separate kernel for kthCounts
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1436-1438
```cpp
1436:   computeBlockwiseKthCounts<Bitwise><<<std::min(((int64_t)numInputSlices + 255) / 256, (int64_t)1073741824), 256, 0, stream>>>(
1437:     desired, counts, num_blocks, blocks_per_slice, kthCounts);
1438:   C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1440-1440
```cpp
1440:   // CUDA path: Do a prefix scan of withinKCounts and kthCounts using slice_idx as keys to get the starting index of each block
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1441-1449
```cpp
1441:   using counting_iter_t = ATEN_CUB_COUNTING_ITERATOR(uint32_t, uint32_t);
1442:   using slice_idx_iter_t = ATEN_CUB_TRANSFORM_ITERATOR(uint32_t, BlockIdxToKey, counting_iter_t);
1443:   slice_idx_iter_t slice_idx_iter(counting_iter_t(0), BlockIdxToKey(blocks_per_slice));
1444:   at::cuda::cub::inclusive_sum_by_key(slice_idx_iter, withinKCounts, withinKCounts, num_blocks);
1445:   at::cuda::cub::inclusive_sum_by_key(slice_idx_iter, kthCounts, kthCounts, num_blocks);
1446: #else
1447:   // ROCm path: kthCounts already produced in computeBlockwiseWithinKCounts at last pass
1448:   // No CUB scans; prefix computation fused into gatherTopK
1449: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1451-1451
```cpp
1451:   // copy topk values to output tensor
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1452-1457
```cpp
1452:   gatherTopK<T, IndexType, Dim><<<grid, block, 0, stream>>>(
1453:     input, inputSliceSize, outputSliceSize, largest, numInputSlices, inputWithinSliceStride,
1454:     topK, topKWithinSliceStride, indices, indicesWithinSliceStride, items_per_thread,
1455:     blocks_per_slice, kthValues, withinKCounts, kthCounts, num_blocks);
1456:   C10_CUDA_KERNEL_LAUNCH_CHECK();
1457: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1459-1459
```cpp
1459: } // namespace mbtopk
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 1461-1472
```cpp
1461: bool should_use_multiblock(int64_t num_slices, int64_t slice_size) {
1462:   if (num_slices > std::numeric_limits<uint32_t>::max() ||
1463:       slice_size > std::numeric_limits<uint32_t>::max()) return false;
1464:   // This heuristics is based on the experiment in https://github.com/pytorch/pytorch/pull/74267
1465:   return (num_slices <= 20 && slice_size >= 20000) ||
1466:       (num_slices > 20 && num_slices <= 40 && slice_size >= 10000) ||
1467:       (num_slices > 40 && num_slices <= 80 && slice_size >= 8000) ||
1468:       (num_slices > 80 && num_slices < 200 && slice_size >= 5000) ||
1469:       (num_slices >= 200 && num_slices < 800 && slice_size >= 3000) ||
1470:       (num_slices >= 800 && num_slices <= 4000 && slice_size >= 800) ||
1471:       (num_slices > 4000 && slice_size >= 400);
1472: }
```
- EN: This block defines or continues the implementation of `should_use_multiblock`.
- CN: 该代码块定义或继续实现 `should_use_multiblock`。

### Lines 1474-1486
```cpp
1474: bool should_use_warp_topk(int64_t slice_size, int64_t k) {
1475: #if !defined(USE_ROCM) || !HAS_WARP_MERGE_SORT()
1476:   return false;
1477: #else
1478:   if (slice_size <= 0 || k <= 0 || slice_size > warptopk::MAX_WARP_TOPK_SLICE) {
1479:     return false;
1480:   }
1481:   // Use WarpMergeSort for small slices
1482:   // WarpMergeSort has O(n log n) complexity and is efficient for small sizes
1483:   // Conservative threshold: slice_size <= 256 shows consistent improvements
1484:   return slice_size <= 256;
1485: #endif
1486: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `should_use_warp_topk`.
- CN: 该代码块定义或继续实现 `should_use_warp_topk`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1488-1509
```cpp
1488: void launch_gather_topk_kernel(
1489:     const TensorBase& self, int64_t k, int64_t dim, bool largest,
1490:     const TensorBase& values, const TensorBase& indices) {
1491:   int numDims = self.dim();
1492:   numDims = numDims == 0 ? 1 : numDims;
1493:   TORCH_CHECK(numDims <= MAX_DIMS, "input tensor has too many dimensions");
1494:   int64_t sliceSize = self.dim() == 0 ? 1 : self.size(dim);
1495: 
1496:   auto input = self.contiguous();
1497:   // static_cast is required to ensure that the correct type (INDEX_T)
1498:   // is provided to the kernel for the arguments.
1499: #define RUN_K(INDEX_T, DIM, LAUNCH_FUNCTION_NAME)                       \
1500:   LAUNCH_FUNCTION_NAME<scalar_t, INDEX_T, DIM>(                         \
1501:       inputInfo,                                                        \
1502:       static_cast<INDEX_T>(sliceSize),                                  \
1503:       static_cast<INDEX_T>(k),                                          \
1504:       largest,                                                          \
1505:       static_cast<INDEX_T>(numInputSlices),                             \
1506:       /* The actual dimension that the k-selection is running in */     \
1507:       /* may have changed from collapseDims() */                        \
1508:       static_cast<INDEX_T>(inputInfo.strides[collapseInputDim]),        \
1509:       topKInfo,                                                         \
```
- EN: This block defines or continues the implementation of `launch_gather_topk_kernel`.
- CN: 该代码块定义或继续实现 `launch_gather_topk_kernel`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 1510-1512
```cpp
1510:       static_cast<INDEX_T>(topKInfo.strides[collapseTopKDim]),          \
1511:       indicesInfo,                                                      \
1512:       static_cast<INDEX_T>(indicesInfo.strides[collapseIndicesDim]));
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1514-1515
```cpp
1514: #if defined(USE_ROCM) && HAS_WARP_MERGE_SORT()
1515: #define RUN_MB(INDEX_T, DIM)                                              \
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1516-1529
```cpp
1516:   if (should_use_warp_topk(sliceSize, k)) {                               \
1517:     RUN_K(INDEX_T, DIM, warptopk::launch);                                \
1518:   } else if (should_use_multiblock(numInputSlices, sliceSize)) {          \
1519:     RUN_K(INDEX_T, DIM, mbtopk::launch);                                  \
1520:   } else {                                                                \
1521:     RUN_K(INDEX_T, DIM, sbtopk::launch);                                  \
1522:   }
1523: #else
1524: #define RUN_MB(INDEX_T, DIM)                                              \
1525:   if (should_use_multiblock(numInputSlices, sliceSize)) {                 \
1526:     RUN_K(INDEX_T, DIM, mbtopk::launch);                                  \
1527:   } else {                                                                \
1528:     RUN_K(INDEX_T, DIM, sbtopk::launch);                                  \
1529:   }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1530-1532
```cpp
1530: #endif
1531: 
1532: #define RUN_DIM(INDEX_T)                        \
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1533-1541
```cpp
1533:   if (allDims == 1) {                           \
1534:     RUN_MB(INDEX_T, 1);                         \
1535:   } else if (allDims == 2) {                    \
1536:     RUN_MB(INDEX_T, 2);                         \
1537:   } else if (allDims == 3) {                    \
1538:     RUN_MB(INDEX_T, 3);                         \
1539:   } else {                                      \
1540:     RUN_MB(INDEX_T, -1);                        \
1541:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1543-1543
```cpp
1543: #define RUN_T(INDEX_T)                                                    \
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1544-1565
```cpp
1544:   AT_DISPATCH_ALL_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, input.scalar_type(), "topk_out_cuda", [&] { \
1545:     at::cuda::detail::TensorInfo<const scalar_t, INDEX_T> inputInfo =     \
1546:       at::cuda::detail::getTensorInfo<const scalar_t, INDEX_T>(input);    \
1547:     at::cuda::detail::TensorInfo<scalar_t, INDEX_T> topKInfo =            \
1548:       at::cuda::detail::getTensorInfo<scalar_t, INDEX_T>(values);         \
1549:     at::cuda::detail::TensorInfo<int64_t, INDEX_T> indicesInfo =          \
1550:       at::cuda::detail::getTensorInfo<int64_t, INDEX_T>(indices);         \
1551:     /* tensorInfoLegacyIfScalar*/                                         \
1552:     if (!input.dim()) {                                                   \
1553:       inputInfo.dims = 1;                                                 \
1554:       inputInfo.sizes[0] = 1;                                             \
1555:       inputInfo.strides[0] = 1;                                           \
1556:       topKInfo.dims = 1;                                                  \
1557:       topKInfo.sizes[0] = 1;                                              \
1558:       topKInfo.strides[0] = 1;                                            \
1559:       indicesInfo.dims = 1;                                               \
1560:       indicesInfo.sizes[0] = 1;                                           \
1561:       indicesInfo.strides[0] = 1;                                         \
1562:     }                                                                     \
1563:     /* We use these structures solely to find the offset to */            \
1564:     /* each slice we are operating on */                                  \
1565:     inputInfo.sizes[dim] = 1;                                             \
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1566-1579
```cpp
1566:     topKInfo.sizes[dim] = 1;                                              \
1567:     indicesInfo.sizes[dim] = 1;                                           \
1568:     /* stash the stride of dim because it can be accidentally collapsed */ \
1569:     auto strideInput = inputInfo.strides[dim];                            \
1570:     auto strideTopK = topKInfo.strides[dim];                              \
1571:     auto strideIndices = indicesInfo.strides[dim];                        \
1572:     /* Collapse all other dims */                                         \
1573:     int collapseInputDim = inputInfo.collapseDims(dim);                   \
1574:     int collapseTopKDim = topKInfo.collapseDims(dim);                     \
1575:     int collapseIndicesDim = indicesInfo.collapseDims(dim);               \
1576:     /* restore stride in case it was collapsed */                         \
1577:     inputInfo.strides[collapseInputDim] = strideInput;                    \
1578:     topKInfo.strides[collapseTopKDim] = strideTopK;                       \
1579:     indicesInfo.strides[collapseIndicesDim] = strideIndices;              \
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1580-1593
```cpp
1580:     int64_t numInputSlices = 1;                                           \
1581:     for (int i = 0; i < inputInfo.dims; ++i) {                            \
1582:       numInputSlices *= inputInfo.sizes[i];                               \
1583:     }                                                                     \
1584:                                                                           \
1585:     /* This is used as a template parameter to calculate indices. */      \
1586:     /* We only specialize it if all collapsed dim sizes are the */        \
1587:     /* same; otherwise, we use -1 which is the specialization */          \
1588:     /* parameter for arbitrary dimensions */                              \
1589:     int allDims = inputInfo.dims;                                         \
1590:     if (topKInfo.dims != allDims || indicesInfo.dims != allDims) {        \
1591:       allDims = -1;                                                       \
1592:     }                                                                     \
1593:                                                                           \
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1594-1595
```cpp
1594:     RUN_DIM(INDEX_T);                                                     \
1595:   });
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1597-1598
```cpp
1597:   // the below is safe with 0-dimensional tensors because it is based on
1598:   // TensorInfo which implicitly expands to 1-dimensional.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1599-1612
```cpp
1599:   if (input.numel() > 0) {
1600:     if (at::cuda::detail::canUse32BitIndexMath(input) &&
1601:         at::cuda::detail::canUse32BitIndexMath(values) &&
1602:         at::cuda::detail::canUse32BitIndexMath(indices)) {
1603:       RUN_T(uint32_t);
1604:     } else {
1605:       RUN_T(uint64_t);
1606:     }
1607:   }
1608: #undef RUN_T
1609: #undef RUN_DIM
1610: #undef RUN_MB
1611: #undef RUN_K
1612: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1614-1614
```cpp
1614: } // at::native
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `CUDA_KERNEL_LOOP` expands index-space iteration on the GPU. / `CUDA_KERNEL_LOOP` 展开 GPU 上的索引空间循环。
- CUB primitives provide parallel CUDA building blocks such as reductions or scans. / CUB 原语提供归约、扫描等并行 CUDA 基础组件。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/native/cuda/TensorTopK.h>`
  - `<ATen/core/TensorBase.h>`
  - `<ATen/ceil_div.h>`
  - `<ATen/Dispatch.h>`
  - `<c10/macros/Macros.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/detail/TensorInfo.cuh>`
  - `<ATen/cuda/detail/OffsetCalculator.cuh>`
  - `<ATen/cuda/ScanUtils.cuh>`
  - `<ATen/cuda/AsmUtils.cuh>`
  - `<ATen/cuda/DeviceUtils.cuh>`
  - `<ATen/native/cuda/SortingCommon.cuh>`
- Runtime symbols / 运行时符号:
  - `launch_gather_topk_kernel`
  - `AT_DISPATCH_ALL_TYPES_AND2`
  - `cub::WarpLoad`
  - `cub::WARP_LOAD_TRANSPOSE`
  - `cub::WarpMergeSort`
  - `cub::WarpStore`
  - `cub::WARP_STORE_TRANSPOSE`
  - `cub::BlockScan`
  - `cub::inclusive_sum_by_key`
  - `at::cuda::detail::TensorInfo`
  - `at::cuda::detail::IndexToOffset`
  - `at::cuda::exclusiveBinaryPrefixScan`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
