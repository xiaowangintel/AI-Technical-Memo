# SortStable.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/SortStable.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `sort_postprocess_kernel`, `segmented_sort_large_segments`, `segmented_sort_pairs_by_full_sort`, `segmented_sort_pairs`.
- 用途（中文）: 实现与 `sort_postprocess_kernel`, `segmented_sort_large_segments`, `segmented_sort_pairs_by_full_sort`, `segmented_sort_pairs` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 2-15
```cpp
   2: #define TORCH_ASSERT_NO_OPERATORS
   3: #include <ATen/native/cuda/SortStable.h>
   4: 
   5: #include <ATen/Dispatch.h>
   6: #include <ATen/core/TensorBase.h>
   7: #include <ATen/cuda/CUDAContext.h>
   8: #include <ATen/cuda/detail/KernelUtils.h>
   9: #include <ATen/cuda/cub.cuh>
  10: #include <ATen/cuda/detail/OffsetCalculator.cuh>
  11: #include <ATen/native/cuda/SortUtils.cuh>
  12: #include <ATen/native/cuda/SortingCommon.cuh>
  13: 
  14: #include <c10/core/DeviceArray.h>
  15: #include <limits>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/SortStable.h>`, `<ATen/Dispatch.h>`, `<ATen/core/TensorBase.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/SortStable.h>`, `<ATen/Dispatch.h>`, `<ATen/core/TensorBase.h>`。

### Lines 17-38
```cpp
  17: namespace at::native {
  18: 
  19: namespace {
  20: 
  21: struct offset_t {
  22:   int stride;
  23:   int begin;
  24:   __host__ __device__ int operator[](int i) const {
  25:     return stride * (begin + i);
  26:   }
  27: #if CCCL_VERSION >= 3001000
  28:   __host__ __device__ offset_t& operator+=(int i) {
  29:     begin += i;
  30:     return *this;
  31:   }
  32: #endif
  33: };
  34: // Segmented sort by full sort algorithm:.
  35: // Say we are sorting a (2, 3) tensor. We have in flattened form:
  36: // values       0.4 1.2 5.3 6.2 1.3 2.3
  37: // indices        0   1   2   0   1   2
  38: // segment_id     0   0   0   1   1   1
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 40-58
```cpp
  40: // First we sort by values, globally:
  41: // values       6.2 5.3 2.3 1.2 1.3 0.4
  42: // indices        0   2   2   1   1   0
  43: // segment_id     1   0   1   0   1   0
  44: 
  45: // Then we stable sort by segment id:
  46: // values       5.3 1.2 0.4 6.2 2.3 1.3
  47: // indices        2   1   0   0   2   1
  48: // segment_id     0   0   0   1   1   1
  49: 
  50: // This method can only work if the slice we are sorting (`dim`) is
  51: // innermost, and both values and indices are contiguous. We do this
  52: // by re-arranging the input into this form as needed, which will
  53: // unfortunately allocate memory if the request is not in this form.
  54: // Vectorized sort is slower than iterated sort if the number of
  55: // slices is small (since we're sorting twice, instead of invoking a
  56: // smaller sort `numSlices` times), but the cub sort
  57: // implementation here is a catch-all, so we're not looking for
  58: // efficiency, but instead correctness.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 60-81
```cpp
  60: template <typename scalar_t>
  61: __global__ void sort_postprocess_kernel(
  62:     const scalar_t* in,
  63:     scalar_t* out,
  64:     int64_t* index,
  65:     const int2* i_s_ptr,
  66:     int nsegments,
  67:     int nsort) {
  68:   CUDA_KERNEL_LOOP(i, nsegments * nsort) {
  69:     int segment = i / nsort;
  70:     int j = i % nsort;
  71: 
  72:     int offset = segment * nsort;
  73:     const scalar_t* in_ = in + offset;
  74:     scalar_t* out_ = out + offset;
  75:     int64_t* index_ = index + offset;
  76:     const int2* i_s_ptr_ = i_s_ptr + offset;
  77: 
  78:     int idx = i_s_ptr_[j].y;
  79:     index_[j] = idx;
  80:     out_[j] = in_[idx];
  81:   }
```
- EN: This block defines GPU kernel entry point(s) `sort_postprocess_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `sort_postprocess_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 82-82
```cpp
  82: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 84-95
```cpp
  84: C10_LAUNCH_BOUNDS_1(at::cuda::detail::CUDA_NUM_THREADS)
  85: __global__ void fill_index_and_segment_kernel(
  86:     int2* data,
  87:     int numel,
  88:     at::cuda::detail::IntDivider<uint32_t> nsort_divider) {
  89:   CUDA_KERNEL_LOOP(idx, numel) {
  90:     auto div_mod = nsort_divider.divmod(idx);
  91:     auto segment = static_cast<int>(div_mod.div);
  92:     auto sort = static_cast<int>(div_mod.mod);
  93:     data[idx] = int2{segment, sort};
  94:   }
  95: }
```
- EN: This block defines GPU kernel entry point(s) `fill_index_and_segment_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `fill_index_and_segment_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 97-105
```cpp
  97: C10_LAUNCH_BOUNDS_1(at::cuda::detail::CUDA_NUM_THREADS)
  98: __global__ void fill_reverse_indices_kernel(
  99:     int64_t* data,
 100:     int numel,
 101:     at::cuda::detail::IntDivider<uint32_t> nsort_divider) {
 102:   CUDA_KERNEL_LOOP(idx, numel) {
 103:     data[idx] = nsort_divider.mod(idx);
 104:   }
 105: }
```
- EN: This block defines GPU kernel entry point(s) `fill_reverse_indices_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `fill_reverse_indices_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 107-128
```cpp
 107: template <typename scalar_t>
 108: inline void segmented_sort_large_segments(
 109:     const int64_t nsegments,
 110:     const int64_t nsort,
 111:     const int64_t n,
 112:     const bool descending,
 113:     const scalar_t* self_ptr,
 114:     scalar_t* values_ptr,
 115:     int64_t* indices_ptr) {
 116:   using namespace at::cuda::detail;
 117:   auto allocator = at::cuda::getCUDADeviceAllocator();
 118:   auto stream = at::cuda::getCurrentCUDAStream();
 119:   dim3 block = CUDA_NUM_THREADS;
 120:   dim3 grid = GET_BLOCKS(nsort);
 121:   c10::DeviceArray<int64_t> indices(*allocator, nsort);
 122:   at::cuda::detail::IntDivider<uint32_t> nsort_divider(nsort);
 123:   fill_reverse_indices_kernel<<<grid, block, 0, stream>>>(
 124:       indices.get(), nsort, nsort_divider);
 125:   const int64_t* initial_indices = indices.get();
 126: 
 127:   for (auto i : c10::irange(nsegments)) {
 128:     at::cuda::cub::radix_sort_pairs<scalar_t, int64_t>(
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `segmented_sort_large_segments`.
- CN: 该代码块定义或继续实现 `segmented_sort_large_segments`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 129-134
```cpp
 129:         self_ptr, values_ptr, initial_indices, indices_ptr, nsort, descending);
 130:     indices_ptr += nsort;
 131:     self_ptr += nsort;
 132:     values_ptr += nsort;
 133:   }
 134: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 136-157
```cpp
 136: template <typename scalar_t>
 137: inline void segmented_sort_pairs_by_full_sort(
 138:     const int64_t nsegments,
 139:     const int64_t nsort,
 140:     const int64_t n,
 141:     const bool descending,
 142:     const scalar_t* const self_ptr,
 143:     scalar_t* const values_ptr,
 144:     int64_t* const indices_ptr) {
 145:   int64_t segment_bits = std::max<int64_t>(
 146:       1L, static_cast<int64_t>(std::ceil(std::log2(nsegments))));
 147: 
 148:   const auto numel = nsort * nsegments;
 149:   auto cuda_allocator = at::cuda::getCUDADeviceAllocator();
 150:   auto indices_and_segment = cuda_allocator->allocate(numel * sizeof(int2));
 151:   auto i_s_ptr = static_cast<int2*>(indices_and_segment.get());
 152: 
 153:   using namespace at::cuda::detail;
 154:   dim3 block = CUDA_NUM_THREADS;
 155:   dim3 grid = GET_BLOCKS(numel);
 156:   auto stream = c10::cuda::getCurrentCUDAStream();
 157:   at::cuda::detail::IntDivider<uint32_t> nsort_divider(nsort);
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `segmented_sort_pairs_by_full_sort`.
- CN: 该代码块定义或继续实现 `segmented_sort_pairs_by_full_sort`。

### Lines 158-159
```cpp
 158:   fill_index_and_segment_kernel<<<grid, block, 0, stream>>>(
 159:       i_s_ptr, numel, nsort_divider);
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 161-163
```cpp
 161:   auto indices_and_segment2 =
 162:       cuda_allocator->allocate(nsegments * nsort * sizeof(int2));
 163:   auto i_s_ptr2 = static_cast<int2*>(indices_and_segment2.get());
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 165-166
```cpp
 165:   at::cuda::cub::radix_sort_pairs<scalar_t, int2>(
 166:       self_ptr, nullptr, i_s_ptr, i_s_ptr2, n, descending);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 168-168
```cpp
 168:   TORCH_INTERNAL_ASSERT(segment_bits <= 32);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 170-170
```cpp
 170:   // sort on lower 32bits, i.e. segment index
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 171-177
```cpp
 171:   at::cuda::cub::radix_sort_keys<int64_t>(
 172:       reinterpret_cast<int64_t*>(i_s_ptr2),
 173:       reinterpret_cast<int64_t*>(i_s_ptr),
 174:       n,
 175:       false,
 176:       0,
 177:       segment_bits);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 179-185
```cpp
 179:   sort_postprocess_kernel<<<
 180:       (n + 511) / 512,
 181:       512,
 182:       0,
 183:       at::cuda::getCurrentCUDAStream()>>>(
 184:       self_ptr, values_ptr, indices_ptr, i_s_ptr, nsegments, nsort);
 185: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 187-207
```cpp
 187: template <typename scalar_t>
 188: void segmented_sort_pairs(
 189:     int64_t nsegments,
 190:     int64_t nsort,
 191:     int64_t n,
 192:     bool descending,
 193:     const scalar_t* self_ptr,
 194:     scalar_t* values_ptr,
 195:     int64_t* indices_ptr) {
 196:   const auto numel = nsort * nsegments;
 197:   auto cuda_allocator = at::cuda::getCUDADeviceAllocator();
 198:   auto reverse_indices = cuda_allocator->allocate(numel * sizeof(int64_t));
 199:   int64_t* reverse_indices_ptr = static_cast<int64_t*>(reverse_indices.get());
 200: 
 201:   using namespace at::cuda::detail;
 202:   dim3 block = CUDA_NUM_THREADS;
 203:   dim3 grid = GET_BLOCKS(numel);
 204:   auto stream = c10::cuda::getCurrentCUDAStream();
 205:   at::cuda::detail::IntDivider<uint32_t> nsort_divider(nsort);
 206:   fill_reverse_indices_kernel<<<grid, block, 0, stream>>>(
 207:       reverse_indices_ptr, numel, nsort_divider);
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `segmented_sort_pairs`.
- CN: 该代码块定义或继续实现 `segmented_sort_pairs`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 209-219
```cpp
 209:   at::cuda::cub::segmented_sort_pairs(
 210:       self_ptr,
 211:       values_ptr,
 212:       reverse_indices_ptr,
 213:       indices_ptr,
 214:       n,
 215:       nsegments,
 216:       offset_t{(int)nsort, 0},
 217:       offset_t{(int)nsort, 1},
 218:       descending);
 219: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 221-221
```cpp
 221: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 223-244
```cpp
 223: void launch_stable_sort_kernel(
 224:     const TensorBase& self,
 225:     int64_t dim,
 226:     bool descending,
 227:     const TensorBase& values,
 228:     const TensorBase& indices) {
 229:   const auto numel = self.numel();
 230:   if (numel == 0) {
 231:     return;
 232:   }
 233: 
 234:   const int64_t intmax = static_cast<int64_t>(std::numeric_limits<int>::max());
 235:   // On ROCm, std::min -> ::min did not work as expected on when input values >= 2147483648
 236:   int64_t numel_or_intmax = numel < intmax ? numel : intmax;
 237:   int64_t nsort = self.size(dim);
 238:   int64_t nbatch = (numel_or_intmax / nsort) * nsort;
 239:   TORCH_CHECK(nbatch > 0, "Cannot sort dimension of length ", nsort);
 240:   int64_t* indices_ptr = indices.mutable_data_ptr<int64_t>();
 241: 
 242:   AT_DISPATCH_ALL_TYPES_AND3(
 243:       kBool, kHalf, kBFloat16, self.scalar_type(), "sort", [&] {
 244:         const scalar_t* self_ptr = self.const_data_ptr<scalar_t>();
```
- EN: This block defines or continues the implementation of `launch_stable_sort_kernel`.
- CN: 该代码块定义或继续实现 `launch_stable_sort_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 245-266
```cpp
 245:         scalar_t* values_ptr = values.mutable_data_ptr<scalar_t>();
 246:         int64_t remaining = numel;
 247:         while (remaining > 0) {
 248:           // On ROCm, std::min -> ::min did not work as expected on when input values >= 2147483648
 249:           int64_t n = remaining < nbatch ? remaining : nbatch;
 250:           int64_t nsegments = n / nsort;
 251: 
 252:           if (nsegments == 1 ||
 253:               nsort >= 1000000) { // rough heuristics where even a single
 254:                                   // sort occupies GPU
 255:             segmented_sort_large_segments(
 256:                 nsegments,
 257:                 nsort,
 258:                 n,
 259:                 descending,
 260:                 self_ptr,
 261:                 values_ptr,
 262:                 indices_ptr);
 263:           } else if (nsegments < 128) {
 264:             segmented_sort_pairs_by_full_sort(
 265:                 nsegments,
 266:                 nsort,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 267-280
```cpp
 267:                 n,
 268:                 descending,
 269:                 self_ptr,
 270:                 values_ptr,
 271:                 indices_ptr);
 272:           } else {
 273:             segmented_sort_pairs(
 274:                 nsegments,
 275:                 nsort,
 276:                 n,
 277:                 descending,
 278:                 self_ptr,
 279:                 values_ptr,
 280:                 indices_ptr);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 281-281
```cpp
 281:           }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 283-289
```cpp
 283:           remaining -= n;
 284:           self_ptr += n;
 285:           values_ptr += n;
 286:           indices_ptr += n;
 287:         }
 288:       });
 289: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 291-291
```cpp
 291: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

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
  - `<ATen/native/cuda/SortStable.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/core/TensorBase.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/detail/KernelUtils.h>`
  - `<ATen/cuda/cub.cuh>`
  - `<ATen/cuda/detail/OffsetCalculator.cuh>`
  - `<ATen/native/cuda/SortUtils.cuh>`
  - `<ATen/native/cuda/SortingCommon.cuh>`
  - `<c10/core/DeviceArray.h>`
  - `<limits>`
- Runtime symbols / 运行时符号:
  - `launch_stable_sort_kernel`
  - `AT_DISPATCH_ALL_TYPES_AND3`
  - `cub::radix_sort_pairs`
  - `cub::radix_sort_keys`
  - `cub::segmented_sort_pairs`
  - `at::cuda::detail::CUDA_NUM_THREADS`
  - `at::cuda::detail::IntDivider`
  - `at::cuda::detail`
  - `at::cuda::getCUDADeviceAllocator`
  - `at::cuda::getCurrentCUDAStream`
  - `at::cuda::cub::radix_sort_pairs`
  - `at::cuda::cub::radix_sort_keys`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
