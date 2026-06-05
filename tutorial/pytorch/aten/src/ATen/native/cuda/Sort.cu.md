# Sort.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/Sort.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `minimum_grid_for_occupancy`, `has_nan`, `constexpr`, `sort`.
- 用途（中文）: 实现与 `minimum_grid_for_occupancy`, `has_nan`, `constexpr`, `sort` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
   1: #define TORCH_ASSERT_NO_OPERATORS
   2: #include <ATen/native/cuda/Sort.h>
   3: #include <ATen/core/TensorBase.h>
   4: #include <ATen/Dispatch.h>
   5: #include <ATen/cuda/cub.cuh>
   6: #include <ATen/cuda/CUDAContext.h>
   7: #include <ATen/cuda/detail/KernelUtils.h>
   8: #include <ATen/cuda/detail/OffsetCalculator.cuh>
   9: #include <ATen/cuda/NumericLimits.cuh>
  10: #include <ATen/native/cuda/SortUtils.cuh>
  11: #include <ATen/native/cuda/SortingCommon.cuh>
  12: 
  13: #include <limits>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/Sort.h>`, `<ATen/core/TensorBase.h>`, `<ATen/Dispatch.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/Sort.h>`, `<ATen/core/TensorBase.h>`, `<ATen/Dispatch.h>`。

### Lines 15-36
```cpp
  15: namespace at::native {
  16: 
  17: template <typename T>
  18: static int minimum_grid_for_occupancy(T kernel, int max_block_size) {
  19:   int minGridSize = 0;
  20:   int blockSize = 0;
  21:   C10_CUDA_CHECK(cudaOccupancyMaxPotentialBlockSize(
  22:       &minGridSize,
  23:       &blockSize,
  24:       kernel,
  25:       /*dynamicSMemSize=*/0,
  26:       max_block_size));
  27:   return minGridSize;
  28: }
  29: 
  30: template <typename T>
  31: constexpr bool has_nan() {
  32:   if constexpr (std::numeric_limits<T>::is_specialized) {
  33:     return std::numeric_limits<T>::has_quiet_NaN;
  34:   } else if constexpr (
  35:       c10::is_complex<T>::value ||
  36:       std::is_same_v<T, c10::BFloat16> ||
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `minimum_grid_for_occupancy`, `has_nan`, `constexpr`.
- CN: 该代码块定义或继续实现 `minimum_grid_for_occupancy`, `has_nan`, `constexpr`。

### Lines 37-40
```cpp
  37:       std::is_same_v<T, c10::Half>) {
  38:     return true;
  39:   }
  40: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 42-44
```cpp
  42: // For very small unstable sorts (n <= 32), use bitonicSortKVInPlace
  43: // which can sort multiple arrays within the same block of threads,
  44: // improving occupancy.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 45-66
```cpp
  45: struct SmallBitonicSort {
  46:   template <int A, typename K, typename V, typename IndexType>
  47:   void sort(
  48:       at::cuda::detail::TensorInfo<K, IndexType> keyInfo,
  49:       IndexType keySlices,
  50:       IndexType keySliceSize,
  51:       IndexType keySliceStride,
  52:       at::cuda::detail::TensorInfo<V, IndexType> valueInfo,
  53:       IndexType valueSliceStride,
  54:       bool descending) {
  55:     constexpr int sort_size = 32;
  56:     constexpr int max_block_y = 16;
  57:     constexpr int items_per_thread = 2;
  58:     static_assert(sort_size % items_per_thread == 0, "");
  59:     constexpr int block_x = sort_size / items_per_thread;
  60: 
  61:     TORCH_INTERNAL_ASSERT(keySliceSize <= sort_size);
  62: 
  63:     // Scale batch size down if the grid would be too small
  64:     const auto min_grid = minimum_grid_for_occupancy(
  65:         bitonicSortKVInPlace<
  66:             A, -1, block_x, max_block_y,
```
- EN: This block defines or continues the implementation of `sort`.
- CN: 该代码块定义或继续实现 `sort`。

### Lines 67-71
```cpp
  67:             K, V, LTOp<K, true>, IndexType>,
  68:         block_x * max_block_y);
  69:     const auto max_batch = std::max(IndexType{1}, keySlices / min_grid);
  70:     const int block_y = std::min(IndexType(max_block_y), max_batch);
  71:     dim3 block(block_x, block_y);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 73-77
```cpp
  73:     dim3 grid;
  74:     const int grid_count = (keySlices + block_y - 1) / block_y;
  75:     TORCH_INTERNAL_ASSERT(getGridFromTiles(grid_count, grid),
  76:                           "Too many slices to sort");
  77:     const auto stream = at::cuda::getCurrentCUDAStream();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 79-100
```cpp
  79:     if (descending) {
  80:       bitonicSortKVInPlace<A, -1, block_x, max_block_y>
  81:         <<<grid, block, 0, stream>>>(
  82:           keyInfo,
  83:           keySlices,
  84:           keySliceSize,
  85:           keySliceStride,
  86:           valueInfo,
  87:           valueSliceStride,
  88:           GTOp<K, true>());
  89:       C10_CUDA_KERNEL_LAUNCH_CHECK();
  90:     } else {
  91:       bitonicSortKVInPlace<A, -1, block_x, max_block_y>
  92:         <<<grid, block, 0, stream>>>(
  93:           keyInfo,
  94:           keySlices,
  95:           keySliceSize,
  96:           keySliceStride,
  97:           valueInfo,
  98:           valueSliceStride,
  99:           LTOp<K, true>());
 100:       C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 101-103
```cpp
 101:     }
 102:   }
 103: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 105-105
```cpp
 105: #if HAS_WARP_MERGE_SORT()
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 107-109
```cpp
 107: // For small sorts (n <= 128) we use warpMergeSortKVInPlace which
 108: // sorts one slice per warp and potentially multiple slices in the
 109: // same block for improved occupancy with large batch sizes.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 110-131
```cpp
 110: template <int sort_size, int WARP_SIZE>
 111: struct WarpMergeSort {
 112: 
 113:   template <int A, typename K, typename V, typename IndexType>
 114:   void sort(
 115:       at::cuda::detail::TensorInfo<K, IndexType> keyInfo,
 116:       IndexType keySlices,
 117:       IndexType keySliceSize,
 118:       IndexType keySliceStride,
 119:       at::cuda::detail::TensorInfo<V, IndexType> valueInfo,
 120:       IndexType valueSliceStride,
 121:       bool descending) {
 122:     constexpr int max_block_y = 16;
 123:     const int block_x = at::cuda::warp_size();
 124: 
 125:     TORCH_INTERNAL_ASSERT(keySliceSize <= sort_size);
 126: 
 127:     // Scale batch size down if the grid would be too small
 128:     const auto min_grid = minimum_grid_for_occupancy(
 129:         warpMergeSortKVInPlace<
 130:             A, -1, sort_size, max_block_y,
 131:             K, V, LTOp<K, true>, IndexType, WARP_SIZE>,
```
- EN: This block defines or continues the implementation of `sort`.
- CN: 该代码块定义或继续实现 `sort`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 132-135
```cpp
 132:         block_x * max_block_y);
 133:     const auto max_batch = std::max(IndexType{1}, keySlices / min_grid);
 134:     const int block_y = std::min(IndexType(max_block_y), max_batch);
 135:     dim3 block(block_x, block_y);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 137-141
```cpp
 137:     dim3 grid;
 138:     const int grid_count = (keySlices + block_y - 1) / block_y;
 139:     TORCH_INTERNAL_ASSERT(getGridFromTiles(grid_count, grid),
 140:                           "Too many slices to sort");
 141:     const auto stream = at::cuda::getCurrentCUDAStream();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 143-164
```cpp
 143:     if (descending) {
 144:       const K invalid_key = at::numeric_limits<K>::lower_bound();
 145:       warpMergeSortKVInPlace<
 146:           A, -1, sort_size, max_block_y,
 147:           K, V, GTOp<K, true>, IndexType, WARP_SIZE>
 148:         <<<grid, block, 0, stream>>>(
 149:           keyInfo,
 150:           keySlices,
 151:           keySliceSize,
 152:           keySliceStride,
 153:           valueInfo,
 154:           valueSliceStride,
 155:           GTOp<K, true>(),
 156:           invalid_key);
 157:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 158:     } else {
 159:       const K invalid_key = []{
 160:         // NAN is sorted after inf
 161:         if constexpr(has_nan<K>()) {
 162:           return K(NAN);
 163:         }
 164:         return at::numeric_limits<K>::upper_bound();
```
- EN: This block defines or continues the implementation of `constexpr`.
- CN: 该代码块定义或继续实现 `constexpr`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 165-178
```cpp
 165:       }();
 166:       warpMergeSortKVInPlace<
 167:           A, -1, sort_size, max_block_y,
 168:           K, V, LTOp<K, true>, IndexType, WARP_SIZE>
 169:         <<<grid, block, 0, stream>>>(
 170:           keyInfo,
 171:           keySlices,
 172:           keySliceSize,
 173:           keySliceStride,
 174:           valueInfo,
 175:           valueSliceStride,
 176:           LTOp<K, true>(),
 177:           invalid_key);
 178:       C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 179-181
```cpp
 179:     }
 180:   }
 181: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 183-183
```cpp
 183: #endif // !HAS_WARP_MERGE_SORT()
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 185-185
```cpp
 185: // For medium sizes (128 < n <= 4096) use radixSortKVInplace.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 186-206
```cpp
 186: struct MediumRadixSort {
 187: 
 188:   template <int A, typename K, typename V, typename IndexType>
 189:   void sort(
 190:       at::cuda::detail::TensorInfo<K, IndexType> keyInfo,
 191:       IndexType keySlices,
 192:       IndexType keySliceSize,
 193:       IndexType keySliceStride,
 194:       at::cuda::detail::TensorInfo<V, IndexType> valueInfo,
 195:       IndexType valueSliceStride,
 196:       bool descending) {
 197: 
 198: #define HANDLE_CASE(SIZE, ITEMS_PER_THREAD)         \
 199:     fixed_size_sort<A, SIZE, ITEMS_PER_THREAD>(     \
 200:         keyInfo,                                    \
 201:         keySlices,                                  \
 202:         keySliceSize,                               \
 203:         keySliceStride,                             \
 204:         valueInfo,                                  \
 205:         valueSliceStride,                           \
 206:         descending)
```
- EN: This block defines or continues the implementation of `sort`.
- CN: 该代码块定义或继续实现 `sort`。

### Lines 208-229
```cpp
 208:     int64_t ceilPowerOf2 = nextHighestPowerOf2(keySliceSize);
 209:     TORCH_INTERNAL_ASSERT(ceilPowerOf2 <= 4096);
 210: #ifdef USE_ROCM
 211:     constexpr int default_ipt = 8;
 212: #else
 213:     constexpr int default_ipt = 32;
 214: #endif
 215:     switch (ceilPowerOf2) {
 216:       case 4096:
 217:         HANDLE_CASE(4096, default_ipt);
 218:         break;
 219:       case 2048:
 220:         HANDLE_CASE(2048, default_ipt);
 221:         break;
 222:       case 1024:
 223:       case 512:
 224:       case 256:
 225:         HANDLE_CASE(1024, default_ipt);
 226:         break;
 227:       case 128:
 228:       case 64:
 229: #if !HAS_WARP_MERGE_SORT()
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 230-243
```cpp
 230:         HANDLE_CASE(128, 4);
 231:         break;
 232: #endif
 233:       case 32:
 234:       case 16:
 235:       case 8:
 236:       case 4:
 237:       case 2:
 238: #if HAS_WARP_MERGE_SORT()
 239:         TORCH_INTERNAL_ASSERT(
 240:             false, "Expected size <= 128 to be handled by a different algorithm");
 241: #else
 242:         HANDLE_CASE(32, 2);
 243: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 244-251
```cpp
 244:         break;
 245:       case 1:
 246:         /* Nothing to do, data already sorted */
 247:         break;
 248:       default:
 249:         TORCH_INTERNAL_ASSERT(false);
 250:     }
 251: #undef HANDLE_CASE
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 253-253
```cpp
 253:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 255-276
```cpp
 255:   template <int A, int sort_size, int items_per_thread,
 256:             typename K, typename V, typename IndexType>
 257:   void fixed_size_sort(
 258:       at::cuda::detail::TensorInfo<K, IndexType> keyInfo,
 259:       IndexType keySlices,
 260:       IndexType keySliceSize,
 261:       IndexType keySliceStride,
 262:       at::cuda::detail::TensorInfo<V, IndexType> valueInfo,
 263:       IndexType valueSliceStride,
 264:       bool descending) {
 265:     static_assert(sort_size % items_per_thread == 0, "");
 266:     constexpr int block = sort_size / items_per_thread;
 267:     dim3 grid;
 268:     TORCH_INTERNAL_ASSERT(getGridFromTiles(keySlices, grid),
 269:                           "Too many slices to sort");
 270: 
 271:     const auto stream = at::cuda::getCurrentCUDAStream();
 272:     radixSortKVInPlace<A, -1, block, items_per_thread>
 273:         <<<grid, block, 0, stream>>>(
 274:           keyInfo,
 275:           keySlices,
 276:           keySliceSize,
```
- EN: This block defines or continues the implementation of `fixed_size_sort`.
- CN: 该代码块定义或继续实现 `fixed_size_sort`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 277-283
```cpp
 277:           keySliceStride,
 278:           valueInfo,
 279:           valueSliceStride,
 280:           descending);
 281:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 282:   }
 283: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 285-306
```cpp
 285: template <typename Sorter>
 286: void sortCommon(Sorter sorter, const TensorBase &key, const TensorBase &value,
 287:                 int dim, bool descending) {
 288:   TORCH_CHECK(key.sizes() == value.sizes(),
 289:               "Key tensor must have same size as value tensor");
 290:   int dims = value.dim();
 291:   TORCH_CHECK(dims <= MAX_DIMS, "value tensor has too many dimensions");
 292:   // if key and value tensors have the same size, we do not need to check both
 293: 
 294:   ptrdiff_t inElements = key.numel();
 295: 
 296:   if (inElements == 0) {
 297:     return;
 298:   }
 299: 
 300:   int64_t keySliceSize = key.size(dim);
 301:   ptrdiff_t keySlices = inElements / keySliceSize;
 302: 
 303: #define HANDLE_SORT_CASE(TYPE, A)                   \
 304:   sorter.template sort<A>(                          \
 305:       keyInfo,                                      \
 306:       (TYPE) keySlices,                             \
```
- EN: This block defines or continues the implementation of `sortCommon`.
- CN: 该代码块定义或继续实现 `sortCommon`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 307-311
```cpp
 307:       (TYPE) keySliceSize,                          \
 308:       (TYPE) keyInfo.strides[collapseKeyDim],       \
 309:       valueInfo,                                    \
 310:       (TYPE) valueInfo.strides[collapseValueDim],   \
 311:       descending)
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 313-316
```cpp
 313:   // The constructed key/value tensor info is used to select the slice
 314:   // we are sorting on a per-block basis
 315:   // The constructed key/value tensor info is used to select the slice
 316:   // we are sorting on a per-block basis
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 317-338
```cpp
 317:   AT_DISPATCH_ALL_TYPES_AND3(at::ScalarType::Half, at::ScalarType::BFloat16, at::ScalarType::Bool, key.scalar_type(), "sortKeyValueInplace", [&]  {
 318:     if (at::cuda::detail::canUse32BitIndexMath(key)) {
 319:       at::cuda::detail::TensorInfo<scalar_t, unsigned int> keyInfo =
 320:         at::cuda::detail::getTensorInfo<scalar_t, unsigned int>(key);
 321:       at::cuda::detail::TensorInfo<int64_t, unsigned int> valueInfo =
 322:         at::cuda::detail::getTensorInfo<int64_t, unsigned int>(value);
 323: 
 324:       auto strideKey = keyInfo.strides[dim];
 325:       keyInfo.sizes[dim] = 1;
 326:       int collapseKeyDim = keyInfo.collapseDims(dim);
 327:       keyInfo.strides[collapseKeyDim] = strideKey;
 328:       auto strideValue = valueInfo.strides[dim];
 329:       valueInfo.sizes[dim]=1;
 330:       int collapseValueDim = valueInfo.collapseDims(dim);
 331:       valueInfo.strides[collapseValueDim] = strideValue;
 332: 
 333:       if (keyInfo.isContiguous()) {
 334:         HANDLE_SORT_CASE(unsigned int, -2);
 335:       } else {
 336:         switch (keyInfo.dims) {
 337:           case 2:
 338:             HANDLE_SORT_CASE(unsigned int, 2);
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 339-344
```cpp
 339:             break;
 340:           default:
 341:             HANDLE_SORT_CASE(unsigned int, -1);
 342:             break;
 343:         }
 344:       }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 346-350
```cpp
 346:     } else {
 347:       at::cuda::detail::TensorInfo<scalar_t, uint64_t> keyInfo =
 348:         at::cuda::detail::getTensorInfo<scalar_t, uint64_t>(key);
 349:       at::cuda::detail::TensorInfo<int64_t, uint64_t> valueInfo =
 350:         at::cuda::detail::getTensorInfo<int64_t, uint64_t>(value);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 352-359
```cpp
 352:       auto strideKey = keyInfo.strides[dim];
 353:       keyInfo.sizes[dim] = 1;
 354:       int collapseKeyDim = keyInfo.collapseDims(dim);
 355:       keyInfo.strides[collapseKeyDim] = strideKey;
 356:       auto strideValue = valueInfo.strides[dim];
 357:       valueInfo.sizes[dim]=1;
 358:       int collapseValueDim = valueInfo.collapseDims(dim);
 359:       valueInfo.strides[collapseValueDim] = strideValue;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 361-361
```cpp
 361:       // int64_t case is rare, just instantiate the generic version
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 362-366
```cpp
 362:       HANDLE_SORT_CASE(uint64_t, -1);
 363:     }
 364:   });
 365: #undef HANDLE_SORT_CASE
 366: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 368-389
```cpp
 368: void sortKeyValueInplace(
 369:     const TensorBase& key,
 370:     const TensorBase& value,
 371:     int64_t dim,
 372:     bool descending,
 373:     bool stable) {
 374:   const auto sort_size = key.size(dim);
 375:   if (sort_size <= 1) {
 376:     return; // Already sorted
 377:   } else if (!stable && sort_size <= 32) {
 378:     // NOTE: Bitonic sort is unstable
 379:     sortCommon(SmallBitonicSort{}, key, value, dim, descending);
 380: #if HAS_WARP_MERGE_SORT()
 381:   } else if (sort_size <= 128) {
 382: #ifdef USE_ROCM
 383:     if (at::cuda::warp_size() == 32) {
 384:       sortCommon(WarpMergeSort<128, 32>{}, key, value, dim, descending);
 385:     }
 386:     else {
 387:       sortCommon(WarpMergeSort<128, 64>{}, key, value, dim, descending);
 388:     }
 389: #else
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `sortKeyValueInplace`.
- CN: 该代码块定义或继续实现 `sortKeyValueInplace`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 390-396
```cpp
 390:     sortCommon(WarpMergeSort<128, C10_WARP_SIZE>{}, key, value, dim, descending);
 391: #endif
 392: #endif
 393:   } else {
 394:     sortCommon(MediumRadixSort{}, key, value, dim, descending);
 395:   }
 396: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 398-398
```cpp
 398: }  // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/native/cuda/Sort.h>`
  - `<ATen/core/TensorBase.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/cuda/cub.cuh>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/detail/KernelUtils.h>`
  - `<ATen/cuda/detail/OffsetCalculator.cuh>`
  - `<ATen/cuda/NumericLimits.cuh>`
  - `<ATen/native/cuda/SortUtils.cuh>`
  - `<ATen/native/cuda/SortingCommon.cuh>`
  - `<limits>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_ALL_TYPES_AND3`
  - `at::cuda::detail::TensorInfo`
  - `at::cuda::getCurrentCUDAStream`
  - `at::cuda::warp_size`
  - `at::cuda::detail::canUse32BitIndexMath`
  - `at::cuda::detail::getTensorInfo`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
