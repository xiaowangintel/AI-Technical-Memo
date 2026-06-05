# SortUtils.cuh — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/SortUtils.cuh`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Declares or defines CUDA helpers/templates associated with `swapVars`, `bitonicSwap`, `bitonicSort`, `bitonicSortKVInPlace`.
- 用途（中文）: 声明或定义与 `swapVars`, `bitonicSwap`, `bitonicSort`, `bitonicSortKVInPlace` 相关的 CUDA 辅助函数/模板。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
   1: #pragma once
   2: #include <c10/macros/Macros.h>
   3: 
   4: #include <ATen/cuda/cub.cuh>
   5: #include <ATen/cuda/detail/TensorInfo.cuh>
   6: #include <ATen/cuda/CUDAContext.h>
   7: #include <ATen/cuda/DeviceUtils.cuh>
   8: #include <ATen/native/cuda/SortingCommon.cuh>
   9: #include <ATen/native/cuda/Sort.h>
  10: #include <ATen/native/StridedRandomAccessor.h>
  11: 
  12: #if defined(USE_ROCM)
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<c10/macros/Macros.h>`, `<ATen/cuda/cub.cuh>`, `<ATen/cuda/detail/TensorInfo.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<c10/macros/Macros.h>`, `<ATen/cuda/cub.cuh>`, `<ATen/cuda/detail/TensorInfo.cuh>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 13-14
```cpp
  13: // ROCm: WarpMergeSort available and tested on ROCm 7.0+
  14: // ROCM_VERSION encoding: MAJOR*10000 + MINOR*100 + PATCH
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 15-16
```cpp
  15: #define HAS_WARP_MERGE_SORT() (ROCM_VERSION >= 70000)
  16: #else
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 17-17
```cpp
  17: // CUDA: WarpMergeSort available since CUDA 11.6
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 18-19
```cpp
  18: #define HAS_WARP_MERGE_SORT() (CUDA_VERSION >= 110600)
  19: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 22-43
```cpp
  22: namespace at::native {
  23: 
  24: template <typename T>
  25: __device__ inline void swapVars(T& t1, T& t2) {
  26:   T tmp = t1;
  27:   t1 = t2;
  28:   t2 = tmp;
  29: }
  30: 
  31: template <typename Comparator, typename K, typename V>
  32: __device__ inline void bitonicSwap(K& kA, V& vA, bool& validA,
  33:                                    K& kB, V& vB, bool& validB,
  34:                                    bool dir,
  35:                                    const Comparator& comp) {
  36:   // Invalid entries always sort to the end
  37:   bool swap = (comp(kA, kB) && validA) || !validB;
  38:   if (swap == dir) {
  39:     swapVars(kA, kB);
  40:     swapVars(vA, vB);
  41:     swapVars(validA, validB);
  42:   }
  43: };
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `swapVars`, `bitonicSwap`.
- CN: 该代码块定义或继续实现 `swapVars`, `bitonicSwap`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 45-66
```cpp
  45: template <int Power2SortSize, typename IndexType, typename Comparator,
  46:           typename K, typename V>
  47: __device__ inline void bitonicSort(K *keys,
  48:                                    V *values,
  49:                                    bool *valid,
  50:                                    const Comparator& comp) {
  51: #if !defined(USE_ROCM)
  52: #pragma unroll
  53: #endif
  54:   for (unsigned int size = 2; size < Power2SortSize; size *= 2) {
  55:     bool flag = ((threadIdx.x & (size / 2)) != 0);
  56: 
  57: #if !defined(USE_ROCM)
  58: #pragma unroll
  59: #endif
  60:     for (unsigned int stride = size / 2; stride > 0; stride /= 2) {
  61: 
  62:       __syncthreads();
  63: 
  64:       unsigned int pos = 2 * threadIdx.x - (threadIdx.x & (stride - 1));
  65:       bitonicSwap<Comparator, K, V>(
  66:         keys[pos], values[pos], valid[pos],
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `bitonicSort`.
- CN: 该代码块定义或继续实现 `bitonicSort`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 67-70
```cpp
  67:         keys[pos + stride], values[pos + stride], valid[pos + stride],
  68:         flag, comp);
  69:     }
  70:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 72-74
```cpp
  72: #if !defined(USE_ROCM)
  73: #pragma unroll
  74: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 75-84
```cpp
  75:   for (unsigned int stride = Power2SortSize / 2; stride > 0; stride /= 2) {
  76: 
  77:     __syncthreads();
  78: 
  79:     unsigned int pos = 2 * threadIdx.x - (threadIdx.x & (stride - 1));
  80:     bitonicSwap<Comparator, K, V>(
  81:       keys[pos], values[pos], valid[pos],
  82:       keys[pos + stride], values[pos + stride], valid[pos + stride],
  83:       false, comp);
  84:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 86-86
```cpp
  86:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 88-88
```cpp
  88: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 90-92
```cpp
  90: // at::cuda::detail::TensorInfo version
  91: // Sorts (key, value) pairs (in different tensors) in-place; i.e.,
  92: // modifies the input `keys` and `values`
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 93-114
```cpp
  93: template <int KeyDims, int ValueDims, int block_dim_x, int max_block_dim_y,
  94:           typename K, typename V, typename Comparator, typename IndexType>
  95: C10_LAUNCH_BOUNDS_1(block_dim_x * max_block_dim_y)
  96: __global__ void
  97: bitonicSortKVInPlace(at::cuda::detail::TensorInfo<K, IndexType> keys,
  98:                      IndexType keySlices,
  99:                      IndexType keySliceSize,
 100:                      IndexType keySliceStride,
 101:                      at::cuda::detail::TensorInfo<V, IndexType> values,
 102:                      IndexType valueSliceStride,
 103:                      Comparator comp) {
 104:   // Find the slice of the tensor that we are sorting
 105:   // NOTE: blockDim.y may be less max_block_dim_y
 106:   const IndexType blockIndex = getLinearBlockId<IndexType>();
 107:   const IndexType linearIndex = blockIndex * blockDim.y + threadIdx.y;
 108: 
 109:   // If the entire block is out of bounds exit early
 110:   if (blockIndex * blockDim.y >= keySlices) {
 111:     return;
 112:   }
 113:   // It's also possible for some rows of a block to be out of bounds
 114:   // but all thread need to run for __syncthreads to work.
```
- EN: This block defines GPU kernel entry point(s) `bitonicSortKVInPlace`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `bitonicSortKVInPlace`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 115-115
```cpp
 115:   const bool row_valid = linearIndex < keySlices;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 117-118
```cpp
 117:   constexpr int items_per_thread = 2;
 118:   constexpr int Power2SortSize = block_dim_x * items_per_thread;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 120-120
```cpp
 120:   // Storage for max_block_dim_y sorts performed in parallel
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 121-123
```cpp
 121:   __shared__ K blockSharedKeys[max_block_dim_y][Power2SortSize];
 122:   __shared__ V blockSharedValues[max_block_dim_y][Power2SortSize];
 123:   __shared__ bool blockSharedValid[max_block_dim_y][Power2SortSize];
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 125-127
```cpp
 125:   auto sharedKeys = blockSharedKeys[threadIdx.y];
 126:   auto sharedValues = blockSharedValues[threadIdx.y];
 127:   auto sharedValid = blockSharedValid[threadIdx.y];
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 129-132
```cpp
 129:   const IndexType keyStartOffset =
 130:     at::cuda::detail::IndexToOffset<K, IndexType, KeyDims>::get(linearIndex, keys);
 131:   const IndexType valueStartOffset =
 132:     at::cuda::detail::IndexToOffset<V, IndexType, ValueDims>::get(linearIndex, values);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 134-134
```cpp
 134:   // Load 2 values per thread into the shared workspace
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 135-135
```cpp
 135:   #pragma unroll
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 136-145
```cpp
 136:   for (int k = 0; k < items_per_thread; ++k) {
 137:     auto idx = threadIdx.x + k * blockDim.x;
 138:     bool valid = row_valid && idx < keySliceSize;
 139: 
 140:     sharedKeys[idx] = valid ?
 141:         keys.data[idx * keySliceStride + keyStartOffset] : K{};
 142:     sharedValues[idx] = valid ?
 143:         values.data[idx * valueSliceStride + valueStartOffset] : V{};
 144:     sharedValid[idx] = valid;
 145:   }
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 147-147
```cpp
 147:   // Sort!
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 148-149
```cpp
 148:   bitonicSort<Power2SortSize, IndexType>(
 149:       sharedKeys, sharedValues, sharedValid, comp);
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 151-153
```cpp
 151:   if (!row_valid) {
 152:     return;
 153:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 155-155
```cpp
 155:   // Store outputs
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 156-156
```cpp
 156:   #pragma unroll
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 157-164
```cpp
 157:   for (int k = 0; k < items_per_thread; ++k) {
 158:     auto idx = threadIdx.x + k * blockDim.x;
 159:     if (idx < keySliceSize) {
 160:       keys.data[idx * keySliceStride + keyStartOffset] = sharedKeys[idx];
 161:       values.data[idx * valueSliceStride + valueStartOffset] = sharedValues[idx];
 162:     }
 163:   }
 164: }
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 166-166
```cpp
 166: #if HAS_WARP_MERGE_SORT()
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 168-175
```cpp
 168: // Note [warp merge sort WARP_SIZE template param]
 169: // ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 170: // warpMergeSortKVInPlace was written assuming C10_WARP_SIZE is a constexpr.
 171: // In torch/headeronly/macros/Macros.h, C10_WARP_SIZE is 32 for CUDA, and on
 172: // ROCm it will be 32 or 64 based on the current compile-time gfx target.
 173: // Ideally, warpSize should be used instead of C10_WARP_SIZE in device code, but
 174: // C10_WARP_SIZE within this kernel has been used as a template parameter for
 175: // some device functions. Therefore, a template param for WARP_SIZE was added.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 176-197
```cpp
 176: template <int KeyDims, int ValueDims, int sort_size, int max_block_dim_y,
 177:           typename K, typename V, typename Comparator, typename IndexType,
 178:           int WARP_SIZE>
 179: C10_LAUNCH_BOUNDS_1(WARP_SIZE * max_block_dim_y)
 180: __global__ void
 181: warpMergeSortKVInPlace(
 182:     at::cuda::detail::TensorInfo<K, IndexType> keys,
 183:     IndexType keySlices,
 184:     IndexType keySliceSize,
 185:     IndexType keySliceStride,
 186:     at::cuda::detail::TensorInfo<V, IndexType> values,
 187:     IndexType valueSliceStride,
 188:     Comparator comp,
 189:     K invalid_key) {
 190:   // Find the slice of the tensor that we are sorting
 191:   // NOTE: blockDim.y may be less max_block_dim_y
 192:   const IndexType blockIndex = getLinearBlockId<IndexType>();
 193:   const IndexType linearIndex = blockIndex * blockDim.y + threadIdx.y;
 194: 
 195:   // If this row is out of bounds exit early
 196:   if (linearIndex >= keySlices) {
 197:     return;
```
- EN: This block defines GPU kernel entry point(s) `warpMergeSortKVInPlace`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `warpMergeSortKVInPlace`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 198-198
```cpp
 198:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 200-203
```cpp
 200:   const IndexType keyStartOffset =
 201:     at::cuda::detail::IndexToOffset<K, IndexType, KeyDims>::get(linearIndex, keys);
 202:   const IndexType valueStartOffset =
 203:     at::cuda::detail::IndexToOffset<V, IndexType, ValueDims>::get(linearIndex, values);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 205-206
```cpp
 205:   K *keys_slice = &keys.data[keyStartOffset];
 206:   V *values_slice = &values.data[valueStartOffset];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 208-209
```cpp
 208:   StridedRandomAccessor<K, IndexType> keys_iter(keys_slice, keySliceStride);
 209:   StridedRandomAccessor<V, IndexType> values_iter(values_slice, valueSliceStride);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 211-211
```cpp
 211:   namespace cub = ROCM_HIPCUB(at_cuda_detail::cub);
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 213-218
```cpp
 213:   CUDA_KERNEL_ASSERT(blockDim.x == WARP_SIZE);
 214:   CUDA_KERNEL_ASSERT(blockDim.y <= max_block_dim_y);
 215:   constexpr int items_per_thread = sort_size / WARP_SIZE;
 216:   static_assert(
 217:       items_per_thread * WARP_SIZE == sort_size,
 218:       "sort_size must be a multiple of WARP_SIZE template param");
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 221-225
```cpp
 221:   using LoadKeys = cub::WarpLoad<K, items_per_thread, cub::WARP_LOAD_TRANSPOSE>;
 222:   using LoadValues = cub::WarpLoad<V, items_per_thread, cub::WARP_LOAD_TRANSPOSE>;
 223:   using Sort = cub::WarpMergeSort<K, items_per_thread, WARP_SIZE, V>;
 224:   using StoreKeys = cub::WarpStore<K, items_per_thread, cub::WARP_STORE_TRANSPOSE>;
 225:   using StoreValues = cub::WarpStore<V, items_per_thread, cub::WARP_STORE_TRANSPOSE>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 227-233
```cpp
 227:   __shared__ union {
 228:     typename LoadKeys::TempStorage load_keys;
 229:     typename LoadValues::TempStorage load_values;
 230:     typename Sort::TempStorage sort;
 231:     typename StoreKeys::TempStorage store_keys;
 232:     typename StoreValues::TempStorage store_values;
 233:   } tmp_storage[max_block_dim_y];
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 235-235
```cpp
 235:   auto& warp_storage = tmp_storage[threadIdx.y];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 237-237
```cpp
 237:   // Load inputs
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 238-239
```cpp
 238:   K local_keys[items_per_thread];
 239:   V local_values[items_per_thread];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 241-245
```cpp
 241:   const auto invalid_value = V{};
 242:   LoadKeys(warp_storage.load_keys).Load(keys_iter, local_keys, keySliceSize, invalid_key);
 243:   WARP_SYNC();
 244:   LoadValues(warp_storage.load_values).Load(values_iter, local_values, keySliceSize, invalid_value);
 245:   WARP_SYNC();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 247-249
```cpp
 247:   // Sort! We use stable sort to ensure that invalid values are never
 248:   // sorted before valid values. In testing it performed the same as
 249:   // .Sort, so there is no down-side.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 250-252
```cpp
 250:   Sort(warp_storage.sort).StableSort(
 251:       local_keys, local_values, comp, keySliceSize, invalid_key);
 252:   WARP_SYNC();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 254-254
```cpp
 254:   // Store outputs
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 255-258
```cpp
 255:   StoreKeys(warp_storage.store_keys).Store(keys_iter, local_keys, keySliceSize);
 256:   WARP_SYNC();
 257:   StoreValues(warp_storage.store_values).Store(values_iter, local_values, keySliceSize);
 258: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 260-260
```cpp
 260: #endif // HAS_WARP_MERGE_SORT()
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 262-282
```cpp
 262: template <int KeyDims, int ValueDims,
 263:           int block_size, int items_per_thread,
 264:           typename K, typename V, typename IndexType>
 265: C10_LAUNCH_BOUNDS_1(block_size)
 266: __global__ void
 267: radixSortKVInPlace(at::cuda::detail::TensorInfo<K, IndexType> keys,
 268:                    IndexType keySlices,
 269:                    IndexType keySliceSize,
 270:                    IndexType keySliceStride,
 271:                    at::cuda::detail::TensorInfo<V, IndexType> values,
 272:                    IndexType valueSliceStride,
 273:                    bool descending) {
 274:   static_assert(block_size > 0, "");
 275: 
 276:   // Find the slice of the tensor that we are sorting
 277:   const IndexType linearIndex = getLinearBlockId<IndexType>();
 278:   // Tiling the slices could have us be out of bounds, if there are a
 279:   // lot of slices to sort
 280:   if (linearIndex >= keySlices) {
 281:     return;
 282:   }
```
- EN: This block defines GPU kernel entry point(s) `radixSortKVInPlace`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `radixSortKVInPlace`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 284-287
```cpp
 284:   const IndexType keyStartOffset =
 285:     at::cuda::detail::IndexToOffset<K, IndexType, KeyDims>::get(linearIndex, keys);
 286:   const IndexType valueStartOffset =
 287:     at::cuda::detail::IndexToOffset<V, IndexType, ValueDims>::get(linearIndex, values);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 289-290
```cpp
 289:   K *keys_slice = &keys.data[keyStartOffset];
 290:   V *values_slice = &values.data[valueStartOffset];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 292-293
```cpp
 292:   StridedRandomAccessor<K, IndexType> keys_iter(keys_slice, keySliceStride);
 293:   StridedRandomAccessor<V, IndexType> values_iter(values_slice, valueSliceStride);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 295-295
```cpp
 295:   namespace cub = ROCM_HIPCUB(at_cuda_detail::cub);
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 297-306
```cpp
 297:   using key_t = typename at::cuda::cub::detail::cuda_type<K>::type;
 298:   using LoadKeys = cub::BlockLoad<K, block_size, items_per_thread,
 299:                                   cub::BlockLoadAlgorithm::BLOCK_LOAD_TRANSPOSE>;
 300:   using LoadValues = cub::BlockLoad<V, block_size, items_per_thread,
 301:                                     cub::BlockLoadAlgorithm::BLOCK_LOAD_TRANSPOSE>;
 302:   using Sort = cub::BlockRadixSort<key_t, block_size, items_per_thread, V>;
 303:   using StoreKeys = cub::BlockStore<K, block_size, items_per_thread,
 304:                                     cub::BLOCK_STORE_TRANSPOSE>;
 305:   using StoreValues = cub::BlockStore<V, block_size, items_per_thread,
 306:                                       cub::BLOCK_STORE_TRANSPOSE>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 308-314
```cpp
 308:   __shared__ union {
 309:     typename LoadKeys::TempStorage load_keys;
 310:     typename LoadValues::TempStorage load_values;
 311:     typename Sort::TempStorage sort;
 312:     typename StoreKeys::TempStorage store_keys;
 313:     typename StoreValues::TempStorage store_values;
 314:   } tmp_storage;
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 316-318
```cpp
 316:   // cub's Block operations operate on a fixed number of items, but the
 317:   // actual slice we are sorting might be smaller. So, we need to make
 318:   // up the difference with keys that will always sort higher.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 319-330
```cpp
 319:   const K invalid_key = [descending] {
 320:     using radix_t = typename cub::Traits<key_t>::UnsignedBits;
 321:     union {
 322:       K key;
 323:       radix_t radix;
 324:     } tmp;
 325:     tmp.radix = descending ?
 326:         cub::Traits<key_t>::LOWEST_KEY :
 327:         cub::Traits<key_t>::MAX_KEY;
 328:     return tmp.key;
 329:   }();
 330:   const V invalid_value = static_cast<V>(0);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 332-332
```cpp
 332:   // Load inputs
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 333-334
```cpp
 333:   K local_keys[items_per_thread];
 334:   V local_values[items_per_thread];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 336-339
```cpp
 336:   LoadKeys(tmp_storage.load_keys).Load(keys_iter, local_keys, keySliceSize, invalid_key);
 337:   __syncthreads();
 338:   LoadValues(tmp_storage.load_values).Load(values_iter, local_values, keySliceSize, invalid_value);
 339:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 341-341
```cpp
 341:   // Sort!
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 342-351
```cpp
 342:   if (descending) {
 343:     Sort(tmp_storage.sort).SortDescending(
 344:         reinterpret_cast<key_t (&)[items_per_thread]>(local_keys),
 345:         local_values);
 346:   } else {
 347:     Sort(tmp_storage.sort).Sort(
 348:         reinterpret_cast<key_t (&)[items_per_thread]>(local_keys),
 349:         local_values);
 350:   }
 351:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 353-353
```cpp
 353:   // Store outputs
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 354-357
```cpp
 354:   StoreKeys(tmp_storage.store_keys).Store(keys_iter, local_keys, keySliceSize);
 355:   __syncthreads();
 356:   StoreValues(tmp_storage.store_values).Store(values_iter, local_values, keySliceSize);
 357: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 359-359
```cpp
 359: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- CUB primitives provide parallel CUDA building blocks such as reductions or scans. / CUB 原语提供归约、扫描等并行 CUDA 基础组件。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<c10/macros/Macros.h>`
  - `<ATen/cuda/cub.cuh>`
  - `<ATen/cuda/detail/TensorInfo.cuh>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/DeviceUtils.cuh>`
  - `<ATen/native/cuda/SortingCommon.cuh>`
  - `<ATen/native/cuda/Sort.h>`
  - `<ATen/native/StridedRandomAccessor.h>`
- Runtime symbols / 运行时符号:
  - `cub::WarpLoad`
  - `cub::WARP_LOAD_TRANSPOSE`
  - `cub::WarpMergeSort`
  - `cub::WarpStore`
  - `cub::WARP_STORE_TRANSPOSE`
  - `cub::detail::cuda_type`
  - `cub::BlockLoad`
  - `cub::BlockLoadAlgorithm::BLOCK_LOAD_TRANSPOSE`
  - `cub::BlockRadixSort`
  - `cub::BlockStore`
  - `cub::BLOCK_STORE_TRANSPOSE`
  - `cub::Traits`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
