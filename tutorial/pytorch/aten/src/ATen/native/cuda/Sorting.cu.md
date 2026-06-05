# Sorting.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/Sorting.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `gatherKthValue`, `gatherMedian`, `KthValueLauncher`, `launch`.
- 用途（中文）: 实现与 `gatherKthValue`, `gatherMedian`, `KthValueLauncher`, `launch` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
   1: #define TORCH_ASSERT_NO_OPERATORS
   2: #include <ATen/native/cuda/Sorting.h>
   3: #include <ATen/core/TensorBase.h>
   4: #include <ATen/ceil_div.h>
   5: #include <ATen/Dispatch.h>
   6: #include <ATen/NumericUtils.h>
   7: #include <c10/macros/Macros.h>
   8: #include <ATen/cuda/CUDAContext.h>
   9: #include <ATen/cuda/detail/TensorInfo.cuh>
  10: #include <ATen/native/cuda/SortingCommon.cuh>
  11: #include <ATen/native/cuda/SortingRadixSelect.cuh>
  12: 
  13: #include <c10/cuda/CUDAStream.h>
  14: 
  15: #include <cassert>
  16: #include <cstdlib>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/Sorting.h>`, `<ATen/core/TensorBase.h>`, `<ATen/ceil_div.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/Sorting.h>`, `<ATen/core/TensorBase.h>`, `<ATen/ceil_div.h>`。

### Lines 18-39
```cpp
  18: namespace at::native {
  19: 
  20: namespace {
  21: 
  22: // Finds the rank k element, and its index, of the values along dimension dim
  23: template <typename scalar_t, typename index_t, int Dim>
  24: __global__ void gatherKthValue(
  25:     cuda::detail::TensorInfo<const scalar_t, index_t> input,
  26:     index_t inputSliceSize,
  27:     index_t k,
  28:     index_t numInputSlices,
  29:     index_t inputWithinSliceStride,
  30:     cuda::detail::TensorInfo<scalar_t, index_t> kthValue,
  31:     cuda::detail::TensorInfo<int64_t, index_t> indices) {
  32:   // smem is used by radixSelect for radix bin counts. Type must be index_t to
  33:   // handle sliceSize > INT_MAX.
  34: #ifndef USE_ROCM
  35:   __shared__ index_t smem[C10_WARP_SIZE]; // one per each warp, up to warp limit
  36: #else
  37:   // Maximum shared memory size for radix select (used in countRadixAggregateCounts): NUM_BUFFERS * MAX_WARPS * RADIX_SIZE.
  38:   // HIP workgroups have at most 1024 threads. Warp size is at least 32 (can be 64 on some
  39:   // architectures), so we use 32 for safety: 2 buffers * (1024/32) warps * 4 radix bins = 256.
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `gatherKthValue`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `gatherKthValue`，它们会直接在 CUDA 线程上执行。

### Lines 40-41
```cpp
  40:   __shared__ index_t smem[256];
  41: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 43-46
```cpp
  43:   index_t slice = getLinearBlockId<index_t>();
  44:   if (slice >= numInputSlices) {
  45:     return;
  46:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 48-48
```cpp
  48:   // Find the start offset for our slice
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 49-54
```cpp
  49:   index_t sliceStartIndex =
  50:       cuda::detail::IndexToOffset<const scalar_t, index_t, Dim>::get(slice, input);
  51:   index_t kthValueSliceStartIndex =
  52:       cuda::detail::IndexToOffset<scalar_t, index_t, Dim>::get(slice, kthValue);
  53:   index_t indicesSliceStartIndex =
  54:       cuda::detail::IndexToOffset<int64_t, index_t, Dim>::get(slice, indices);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 56-58
```cpp
  56:   const scalar_t* inputSliceStart = &input.data[sliceStartIndex];
  57:   scalar_t* kthValueSliceStart = &kthValue.data[kthValueSliceStartIndex];
  58:   int64_t* indicesSliceStart = &indices.data[indicesSliceStartIndex];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 60-60
```cpp
  60:   // Find the k-th highest element in our input
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 61-72
```cpp
  61:   scalar_t kValue = static_cast<scalar_t>(0);
  62:   radixSelect<
  63:       scalar_t,
  64:       typename TopKTypeConfig<scalar_t>::RadixType,
  65:       index_t>(
  66:       inputSliceStart,
  67:       k,
  68:       false,
  69:       inputSliceSize,
  70:       inputWithinSliceStride,
  71:       smem,
  72:       &kValue);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 74-74
```cpp
  74:   // Find the index of the k-th highest element
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 75-75
```cpp
  75:   __shared__ int32_t minIndexFound;
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 77-80
```cpp
  77:   if (threadIdx.x == 0) {
  78:       minIndexFound = static_cast<int32_t>(inputSliceSize);
  79:   }
  80:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 82-96
```cpp
  82:   for (index_t i = threadIdx.x; i < inputSliceSize; i += blockDim.x) {
  83:       // Early exit based on best-so-far
  84:       if (i >= minIndexFound) {
  85:           break;
  86:       }
  87: 
  88:       scalar_t v = doLdg(&inputSliceStart[i * inputWithinSliceStride]);
  89:       bool isKValue =
  90:           ((v == kValue) || (at::_isnan(v) && at::_isnan(kValue)));
  91: 
  92:       if (isKValue) {
  93:           atomicMin(&minIndexFound, static_cast<int32_t>(i));
  94:           break;
  95:       }
  96:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 98-98
```cpp
  98:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 100-104
```cpp
 100:   if (threadIdx.x == 0) {
 101:       indicesSliceStart[0] = static_cast<index_t>(minIndexFound);
 102:       kthValueSliceStart[0] = kValue;
 103:   }
 104: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 106-106
```cpp
 106: // CUDA kernel to find the median, and its index, of the values along dimension dim
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 107-128
```cpp
 107: template <typename scalar_t, typename index_t, int Dim>
 108: __global__ void gatherMedian(
 109:     cuda::detail::TensorInfo<scalar_t, index_t> values,
 110:     cuda::detail::TensorInfo<int64_t, index_t> indices,
 111:     cuda::detail::TensorInfo<const scalar_t, index_t> input,
 112:     index_t inputSliceSize,
 113:     index_t numInputSlices,
 114:     index_t inputWithinSliceStride,
 115:     bool ignore_nan) {
 116:   // smem is used by radixSelect for radix bin counts. Type must be index_t to
 117:   // handle sliceSize > INT_MAX.
 118: #ifndef USE_ROCM
 119:   __shared__ index_t smem[C10_WARP_SIZE]; // one per each warp, up to warp limit
 120: #else
 121:   // Maximum shared memory size for radix select (used in countRadixAggregateCounts): NUM_BUFFERS * MAX_WARPS * RADIX_SIZE.
 122:   // HIP workgroups have at most 1024 threads. Warp size is at least 32 (can be 64 on some
 123:   // architectures), so we use 32 for safety: 2 buffers * (1024/32) warps * 4 radix bins = 256.
 124:   __shared__ index_t smem[256];
 125: #endif
 126: 
 127:   index_t slice = getLinearBlockId<index_t>();
 128:   if (slice >= numInputSlices) {
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines GPU kernel entry point(s) `gatherMedian`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `gatherMedian`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 129-130
```cpp
 129:     return;
 130:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 132-132
```cpp
 132:   // Finds the start offset for our slice
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 133-138
```cpp
 133:   index_t valuesSliceStartIndex =
 134:       cuda::detail::IndexToOffset<scalar_t, index_t, Dim>::get(slice, values);
 135:   index_t indicesSliceStartIndex =
 136:       cuda::detail::IndexToOffset<int64_t, index_t, Dim>::get(slice, indices);
 137:   index_t inputSliceStartIndex =
 138:       cuda::detail::IndexToOffset<const scalar_t, index_t, Dim>::get(slice, input);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 140-142
```cpp
 140:   scalar_t* valuesSliceStart = &values.data[valuesSliceStartIndex];
 141:   int64_t* indicesSliceStart = &indices.data[indicesSliceStartIndex];
 142:   const scalar_t* inputSliceStart = &input.data[inputSliceStartIndex];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 144-148
```cpp
 144:   index_t nan_count = 0;
 145:   for (index_t i = threadIdx.x; i < inputSliceSize; i += blockDim.x) {
 146:     scalar_t val = doLdg(&inputSliceStart[i * inputWithinSliceStride]);
 147:     nan_count += at::_isnan(val) ? 1 : 0;
 148:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 150-151
```cpp
 150:   // Counts number of nan values
 151:   // This code performs a parallel sum reduction (not the most efficient code)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 152-160
```cpp
 152:   __shared__ int64_t num_nan;
 153:   if (threadIdx.x == 0) {
 154:     num_nan = 0;
 155:   }
 156:   __syncthreads();
 157:   if (nan_count > 0) {
 158:     gpuAtomicAddNoReturn(&num_nan, nan_count);
 159:   }
 160:   __syncthreads();
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 162-163
```cpp
 162:   // For torch.median, if we found nan set k to last index so the computed value
 163:   // is nan, otherwise set k to the middle element of the non-nan values
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 164-165
```cpp
 164:   index_t k = (!ignore_nan && num_nan > 0) ? inputSliceSize - 1
 165:                                            : (inputSliceSize - num_nan - 1) / 2;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 167-167
```cpp
 167:   // Find the median
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 168-179
```cpp
 168:   scalar_t median = static_cast<scalar_t>(0);
 169:   radixSelect<
 170:       scalar_t,
 171:       typename TopKTypeConfig<scalar_t>::RadixType,
 172:       index_t>(
 173:       inputSliceStart,
 174:       k + 1,
 175:       false,
 176:       inputSliceSize,
 177:       inputWithinSliceStride,
 178:       smem,
 179:       &median);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 181-181
```cpp
 181:   valuesSliceStart[0] = median;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 183-183
```cpp
 183:   // Find the index of the median value in the slice
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 184-191
```cpp
 184:   for (index_t i = threadIdx.x; i < inputSliceSize; i += blockDim.x) {
 185:     scalar_t val = doLdg(&inputSliceStart[i * inputWithinSliceStride]);
 186:     if (val == median || (at::_isnan(val) && at::_isnan(median))) {
 187:       indicesSliceStart[0] = i;
 188:       break;
 189:     }
 190:   }
 191: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 193-214
```cpp
 193: struct KthValueLauncher {
 194:   int64_t k;
 195: 
 196:   KthValueLauncher(int64_t k) : k(k) {}
 197: 
 198:   template <typename scalar_t, typename index_t, int all_dims>
 199:   inline void launch(
 200:       cuda::detail::TensorInfo<scalar_t, index_t> values_info,
 201:       int collapse_values_dim,
 202:       cuda::detail::TensorInfo<int64_t, index_t> indices_info,
 203:       [[maybe_unused]] int collapse_indices_dim,
 204:       cuda::detail::TensorInfo<const scalar_t, index_t> self_info,
 205:       int collapse_self_dim,
 206:       int64_t num_slices,
 207:       int64_t slice_size) {
 208:     dim3 grid;
 209:     if (!getGridFromTiles(num_slices, grid)) {
 210:       TORCH_CHECK(false, "slices are too many");
 211:     }
 212: 
 213:     dim3 block(std::min(
 214:         round_up(slice_size, (int64_t)at::cuda::warp_size()), (int64_t)1024));
```
- EN: This block defines or continues the implementation of `KthValueLauncher`, `launch`.
- CN: 该代码块定义或继续实现 `KthValueLauncher`, `launch`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 215-228
```cpp
 215:     auto stream = at::cuda::getCurrentCUDAStream();
 216:     gatherKthValue<scalar_t, index_t, all_dims><<<grid, block, 0, stream>>>(
 217:         self_info,
 218:         slice_size,
 219:         k,
 220:         num_slices,
 221:         /* The actual dimension that the k-selection is running in */
 222:         /* may have changed from collapseDims() */
 223:         self_info.strides[collapse_self_dim],
 224:         values_info,
 225:         indices_info);
 226:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 227:   }
 228: };
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 230-251
```cpp
 230: struct MedianLauncher {
 231:   bool ignore_nan;
 232: 
 233:   MedianLauncher(bool ignore_nan) : ignore_nan(ignore_nan) {}
 234: 
 235:   template <typename scalar_t, typename index_t, int all_dims>
 236:   inline void launch(
 237:       cuda::detail::TensorInfo<scalar_t, index_t> values_info,
 238:       [[maybe_unused]] int collapse_values_dim,
 239:       cuda::detail::TensorInfo<int64_t, index_t> indices_info,
 240:       [[maybe_unused]] int collapse_indices_dim,
 241:       cuda::detail::TensorInfo<const scalar_t, index_t> self_info,
 242:       int collapse_self_dim,
 243:       int64_t num_slices,
 244:       int64_t slice_size) {
 245:     dim3 grid;
 246:     if (!getGridFromTiles(num_slices, grid)) {
 247:       TORCH_CHECK(false, "slices are too many");
 248:     }
 249: 
 250:     dim3 block(std::min(
 251:         round_up(slice_size, (int64_t)at::cuda::warp_size()), (int64_t)1024));
```
- EN: This block defines or continues the implementation of `MedianLauncher`, `launch`.
- CN: 该代码块定义或继续实现 `MedianLauncher`, `launch`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 252-263
```cpp
 252:     auto stream = at::cuda::getCurrentCUDAStream();
 253:     gatherMedian<scalar_t, index_t, all_dims><<<grid, block, 0, stream>>>(
 254:         values_info,
 255:         indices_info,
 256:         self_info,
 257:         slice_size,
 258:         num_slices,
 259:         self_info.strides[collapse_self_dim],
 260:         ignore_nan);
 261:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 262:   }
 263: };
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 265-265
```cpp
 265: }  // namespace (anonymous)
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 267-281
```cpp
 267: void launch_kthvalue_kernel(
 268:     const TensorBase &values, const TensorBase &indices,
 269:     const TensorBase &self, int64_t dim, int64_t k) {
 270:   AT_DISPATCH_ALL_TYPES_AND2(
 271:       at::ScalarType::Half, at::ScalarType::BFloat16, self.scalar_type(), "kthvalue_cuda", [&] {
 272:     AT_DISPATCH_INDEX_TYPES(
 273:         cuda::detail::canUse32BitIndexMath(self) &&
 274:         cuda::detail::canUse32BitIndexMath(values) &&
 275:         cuda::detail::canUse32BitIndexMath(indices) ? ScalarType::Int : ScalarType::Long,
 276:         "kth_value_launcher", [&] {
 277:           run_launcher<scalar_t, index_t>(
 278:               values, indices, self, dim, KthValueLauncher(k));
 279:     });
 280:   });
 281: }
```
- EN: This block defines or continues the implementation of `launch_kthvalue_kernel`.
- CN: 该代码块定义或继续实现 `launch_kthvalue_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 283-298
```cpp
 283: void launch_median_kernel(
 284:     const TensorBase &vals, const TensorBase &inds,
 285:     const TensorBase &self, int64_t dim, bool ignore_nan) {
 286:   AT_DISPATCH_ALL_TYPES_AND2(
 287:       at::ScalarType::Half, at::ScalarType::BFloat16, self.scalar_type(), "median_out_impl", [&] {
 288:         if (cuda::detail::canUse32BitIndexMath(vals) &&
 289:             cuda::detail::canUse32BitIndexMath(inds) &&
 290:             cuda::detail::canUse32BitIndexMath(self)) {
 291:           run_launcher<scalar_t, uint32_t>(
 292:               vals, inds, self, dim, MedianLauncher(ignore_nan));
 293:         } else {
 294:           run_launcher<scalar_t, uint64_t>(
 295:               vals, inds, self, dim, MedianLauncher(ignore_nan));
 296:         }
 297:       });
 298: }
```
- EN: This block defines or continues the implementation of `launch_median_kernel`.
- CN: 该代码块定义或继续实现 `launch_median_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 300-300
```cpp
 300: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/native/cuda/Sorting.h>`
  - `<ATen/core/TensorBase.h>`
  - `<ATen/ceil_div.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/NumericUtils.h>`
  - `<c10/macros/Macros.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/detail/TensorInfo.cuh>`
  - `<ATen/native/cuda/SortingCommon.cuh>`
  - `<ATen/native/cuda/SortingRadixSelect.cuh>`
  - `<c10/cuda/CUDAStream.h>`
  - `<cassert>`
- Runtime symbols / 运行时符号:
  - `launch_kthvalue_kernel`
  - `launch_median_kernel`
  - `AT_DISPATCH_ALL_TYPES_AND2`
  - `AT_DISPATCH_INDEX_TYPES`
  - `at::cuda::warp_size`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
