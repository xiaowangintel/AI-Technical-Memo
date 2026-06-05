# UniqueCub.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/UniqueCub.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `adjacent_difference_kernel`, `scatter_kernel`, `wrap_input_iterator`, `compute_unique`.
- 用途（中文）: 实现与 `adjacent_difference_kernel`, `scatter_kernel`, `wrap_input_iterator`, `compute_unique` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/native/cuda/UniqueCub.cuh>
   3: 
   4: #include <ATen/cuda/CUDAContext.h>
   5: #include <ATen/cuda/detail/KernelUtils.h>
   6: #include <ATen/cuda/CUDAApplyUtils.cuh>
   7: #include <ATen/cuda/cub.cuh>
   8: 
   9: #include <c10/core/DeviceArray.h>
  10: #include <c10/util/Load.h>
  11: 
  12: #ifndef AT_PER_OPERATOR_HEADERS
  13: #include <ATen/Functions.h>
  14: #else
  15: #include <ATen/ops/arange.h>
  16: #include <ATen/ops/empty.h>
  17: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/UniqueCub.cuh>`, `<ATen/cuda/CUDAContext.h>`, `<ATen/cuda/detail/KernelUtils.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/UniqueCub.cuh>`, `<ATen/cuda/CUDAContext.h>`, `<ATen/cuda/detail/KernelUtils.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 19-40
```cpp
  19: namespace at::native::internal {
  20: 
  21: namespace {
  22: 
  23: template <typename InputIteratorT>
  24: __global__ void adjacent_difference_kernel(
  25:     int64_t n,
  26:     InputIteratorT input,
  27:     int* output) {
  28:   CUDA_KERNEL_LOOP(i, n) {
  29:     output[i] = i > 0 ? input[i] != input[i - 1] : 0;
  30:   }
  31: }
  32: 
  33: __global__ void scatter_kernel(
  34:     int64_t n,
  35:     const int64_t* input,
  36:     const int64_t* indices,
  37:     int64_t* output) {
  38:   CUDA_KERNEL_LOOP(i, n) {
  39:     output[indices[i]] = input[i];
  40:   }
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `adjacent_difference_kernel`, `scatter_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `adjacent_difference_kernel`, `scatter_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 41-41
```cpp
  41: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 43-46
```cpp
  43: template <typename scalar_t>
  44: const scalar_t * wrap_input_iterator(const scalar_t *data) {
  45:   return data;
  46: }
```
- EN: This block defines or continues the implementation of `wrap_input_iterator`.
- CN: 该代码块定义或继续实现 `wrap_input_iterator`。

### Lines 48-52
```cpp
  48: struct LoadBoolOp {
  49:   __device__ bool operator()(uint8_t x) const {
  50:     return static_cast<bool>(x);
  51:   }
  52: };
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 54-59
```cpp
  54: auto wrap_input_iterator(const bool *data) {
  55:   // See NOTE [Loading boolean values]
  56:   LoadBoolOp op;
  57:   return ATEN_CUB_TRANSFORM_ITERATOR(bool, LoadBoolOp, const uint8_t*, int)(
  58:       reinterpret_cast<const uint8_t*>(data), op);
  59: }
```
- EN: This block defines or continues the implementation of `wrap_input_iterator`.
- CN: 该代码块定义或继续实现 `wrap_input_iterator`。

### Lines 61-62
```cpp
  61: // A variation of compute_unique (defined in Unique.cu) that doesn't allow
  62: // customizing equal and not_equal (CUB doesn't allow them).
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 63-84
```cpp
  63: template <typename scalar_t>
  64: std::tuple<Tensor, Tensor, Tensor> compute_unique(
  65:     const Tensor& sorted,
  66:     const Tensor& sorted_indices,
  67:     const bool return_inverse,
  68:     const bool return_counts,
  69:     const bool consecutive) {
  70:   int64_t num_inp = sorted.numel();
  71:   auto options = sorted.options().dtype(kLong);
  72:   auto data = wrap_input_iterator(sorted.const_data_ptr<scalar_t>());
  73:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
  74: 
  75:   // inverse indices
  76:   Tensor inverse_indices;
  77:   if (!return_inverse) {
  78:     inverse_indices = at::empty({0}, options);
  79:   } else {
  80:     inverse_indices = at::empty(sorted.sizes(), options);
  81:     Tensor inv_loc = consecutive ? at::empty({num_inp}, options.dtype(kInt))
  82:                                  : inverse_indices;
  83:     int* inv_loc_ptr = static_cast<int*>(inv_loc.mutable_data_ptr());
  84:     const dim3 block =
```
- EN: This block defines or continues the implementation of `compute_unique`.
- CN: 该代码块定义或继续实现 `compute_unique`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 85-92
```cpp
  85:         dim3(std::min(static_cast<int64_t>(cuda::getApplyBlock().x), num_inp));
  86:     dim3 grid;
  87:     c10::DeviceIndex curDevice = -1;
  88:     C10_CUDA_CHECK(c10::cuda::GetDevice(&curDevice));
  89:     cuda::getApplyGrid(num_inp, grid, curDevice);
  90:     adjacent_difference_kernel<<<grid, block, 0, stream>>>(
  91:         num_inp, data, inv_loc_ptr);
  92:     C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 94-99
```cpp
  94:     Tensor inv_loc_out =
  95:         consecutive ? inverse_indices : at::empty({num_inp}, options);
  96:     at::cuda::cub::inclusive_sum_truncating(
  97:         inv_loc_ptr,
  98:         inv_loc_out.mutable_data_ptr<int64_t>(),
  99:         num_inp);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 101-112
```cpp
 101:     if (!consecutive) {
 102:       TORCH_INTERNAL_ASSERT(
 103:           sorted_indices.defined(),
 104:           "return_inverse is set to true, but sorted_indices is undefined. Send a bug report!");
 105:       scatter_kernel<<<grid, block, 0, stream>>>(
 106:           num_inp,
 107:           inv_loc_out.const_data_ptr<int64_t>(),
 108:           sorted_indices.const_data_ptr<int64_t>(),
 109:           inverse_indices.mutable_data_ptr<int64_t>());
 110:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 111:     }
 112:   }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 114-114
```cpp
 114:   // unique and count
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 115-132
```cpp
 115:   Tensor data_out = at::empty({num_inp}, sorted.options());
 116:   Tensor counts = at::empty({0}, options);
 117:   Tensor length = at::empty({1}, options);
 118:   int64_t num_out;
 119:   if (!return_counts) {
 120:     cuda::cub::unique(data, data_out.mutable_data_ptr<scalar_t>(), length.mutable_data_ptr<int64_t>(), num_inp);
 121:     num_out = length.item<int64_t>();
 122:   } else {
 123:     counts.resize_(num_inp);
 124:     at::cuda::cub::run_length_encode(
 125:         data,
 126:         data_out.mutable_data_ptr<scalar_t>(),
 127:         counts.mutable_data_ptr<int64_t>(),
 128:         length.mutable_data_ptr<int64_t>(),
 129:         num_inp);
 130:     num_out = length.item<int64_t>();
 131:     counts.resize_(num_out);
 132:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 134-137
```cpp
 134:   data_out.resize_(num_out);
 135:   return std::tuple<Tensor, Tensor, Tensor>(
 136:       data_out, inverse_indices, counts);
 137: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 139-139
```cpp
 139: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 141-143
```cpp
 141: // This function (and compute_unique above) are defined in a separate file from
 142: // Unique.cu because for now ATen/cuda/cub.cuh can't be used together with
 143: // thrust in the same compilation unit.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 145-166
```cpp
 145: template <typename scalar_t>
 146: struct UniqueCub {
 147:   std::tuple<Tensor, Tensor, Tensor> operator() (
 148:       const Tensor& self,
 149:       const bool consecutive,
 150:       const bool return_inverse,
 151:       const bool return_counts) {
 152:     cudaStream_t stream = at::cuda::getCurrentCUDAStream();
 153: 
 154:     int64_t num_inp = self.numel();
 155:     Tensor sorted;
 156:     if (consecutive) {
 157:       sorted = self;
 158:     } else {
 159:       sorted = at::empty(self.sizes(), self.options());
 160:     }
 161: 
 162:     Tensor sorted_indices;
 163:     if (!return_inverse) {
 164:       if (!consecutive) {
 165:         cuda::cub::radix_sort_keys(
 166:           self.const_data_ptr<scalar_t>(),
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 167-180
```cpp
 167:           sorted.mutable_data_ptr<scalar_t>(),
 168:           num_inp);
 169:       }
 170:     } else {
 171:       if (!consecutive) {
 172:         auto options = self.options().dtype(kLong);
 173:         Tensor range = at::arange(0, num_inp, options);
 174:         sorted_indices = at::empty({num_inp}, options);
 175:         cuda::cub::radix_sort_pairs(
 176:             self.const_data_ptr<scalar_t>(),
 177:             sorted.mutable_data_ptr<scalar_t>(),
 178:             range.const_data_ptr<int64_t>(),
 179:             sorted_indices.mutable_data_ptr<int64_t>(),
 180:             num_inp);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 181-182
```cpp
 181:       }
 182:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 184-187
```cpp
 184:     return compute_unique<scalar_t>(
 185:         sorted, sorted_indices, return_inverse, return_counts, consecutive);
 186:   }
 187: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 189-193
```cpp
 189: struct MapNumberOfTrueValues {
 190:   __device__ int operator()(uint8_t x) const {
 191:     return static_cast<bool>(x);
 192:   }
 193: };
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 195-210
```cpp
 195: C10_LAUNCH_BOUNDS_1(at::cuda::detail::CUDA_NUM_THREADS)
 196: __global__ void unique_bool_write_inverse_indices(
 197:     const int numel,
 198:     const int *num_true_p,
 199:     const bool *self,
 200:     int64_t *inverse_indices_out) {
 201:   constexpr int false_idx = 0;
 202:   const int num_true = *num_true_p;
 203:   const int num_false = numel - num_true;
 204:   const int true_idx = num_false > 0;
 205: 
 206:   CUDA_KERNEL_LOOP(i, numel) {
 207:     const auto value = c10::load(&self[i]);
 208:     inverse_indices_out[i] = value ? true_idx : false_idx;
 209:   }
 210: }
```
- EN: This block defines GPU kernel entry point(s) `unique_bool_write_inverse_indices`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `unique_bool_write_inverse_indices`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 212-233
```cpp
 212: C10_LAUNCH_BOUNDS_1(1)
 213: __global__ void unique_bool_write_output(
 214:     const int numel,
 215:     const int *num_true_p,
 216:     bool *values_out,
 217:     int64_t *counts_out) {
 218:   constexpr int false_idx = 0;
 219:   const int num_true = *num_true_p;
 220:   const int num_false = numel - num_true;
 221:   const int true_idx = num_false > 0;
 222: 
 223:   if (blockIdx.x == 0 && threadIdx.x == 0) {
 224:     if (num_false > 0) {
 225:       values_out[false_idx] = false;
 226:       counts_out[false_idx] = num_false;
 227:     }
 228:     if (num_true > 0) {
 229:       values_out[true_idx] = true;
 230:       counts_out[true_idx] = num_true;
 231:     }
 232:   }
 233: }
```
- EN: This block defines GPU kernel entry point(s) `unique_bool_write_output`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `unique_bool_write_output`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 235-256
```cpp
 235: template <>
 236: struct UniqueCub<bool> {
 237: 
 238:   std::tuple<Tensor, Tensor, Tensor> operator() (
 239:       const Tensor& self,
 240:       const bool consecutive,
 241:       const bool return_inverse,
 242:       const bool return_counts) {
 243:     auto stream = at::cuda::getCurrentCUDAStream();
 244: 
 245:     int64_t num_inp = self.numel();
 246: 
 247:     Tensor output, inverse_indices, counts;
 248:     if (consecutive) {
 249:       Tensor sorted_indices;
 250:       return compute_unique<bool>(
 251:           self, sorted_indices, return_inverse, return_counts, consecutive);
 252:     }
 253: 
 254:     // Instead of sorting, we use a reduction to find the number of
 255:     // true values and from that we can infer the number of false.
 256:     // If either has a count of zero, we omit it from the output.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 257-258
```cpp
 257:     auto allocator = at::cuda::getCUDADeviceAllocator();
 258:     c10::DeviceArray<int> tmp_num_true(*allocator, 1);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 260-265
```cpp
 260:     const bool* self_data = self.const_data_ptr<bool>();
 261:     MapNumberOfTrueValues op;
 262:     ATEN_CUB_TRANSFORM_ITERATOR(int, MapNumberOfTrueValues, const uint8_t*, int)
 263:         data_iter(reinterpret_cast<const uint8_t*>(self_data), op);
 264:     at::cuda::cub::reduce(data_iter, tmp_num_true.get(), num_inp,
 265:                           NO_ROCM(::cuda)::std::plus<>{}, 0);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 267-269
```cpp
 267:     auto options = self.options();
 268:     output = at::empty({2}, self.options());
 269:     counts = at::empty({2}, options.dtype(kLong));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 271-276
```cpp
 271:     unique_bool_write_output<<<1, 1, 0, stream>>>(
 272:         num_inp,
 273:         tmp_num_true.get(),
 274:         output.mutable_data_ptr<bool>(),
 275:         counts.mutable_data_ptr<int64_t>());
 276:     C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 278-289
```cpp
 278:     if (return_inverse) {
 279:       using namespace at::cuda::detail;
 280:       inverse_indices = at::empty(self.sizes(), options.dtype(kLong));
 281:       dim3 block = CUDA_NUM_THREADS;
 282:       dim3 grid = GET_BLOCKS(num_inp);
 283:       unique_bool_write_inverse_indices<<<grid, block, 0, stream>>>(
 284:           num_inp,
 285:           tmp_num_true.get(),
 286:           self_data,
 287:           inverse_indices.mutable_data_ptr<int64_t>());
 288:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 289:     }
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 291-291
```cpp
 291:     // Final sync to fix the output tensors shape
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 292-298
```cpp
 292:     int num_true = 0;
 293:     at::cuda::memcpy_and_sync(&num_true, tmp_num_true.get(), sizeof(int),
 294:                               cudaMemcpyDeviceToHost, stream);
 295:     const int num_false = num_inp - num_true;
 296:     const int num_out = ((num_true > 0) + (num_false > 0));
 297:     output.resize_({num_out});
 298:     counts.resize_({num_out});
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 300-302
```cpp
 300:     return std::tuple<Tensor, Tensor, Tensor>(output, inverse_indices, counts);
 301:   }
 302: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 304-322
```cpp
 304: template <typename scalar_t>
 305: std::tuple<Tensor, Tensor, Tensor> unique_cuda_template(
 306:     const Tensor& self,
 307:     const bool consecutive,
 308:     const bool return_inverse,
 309:     const bool return_counts) {
 310:   auto num_inp = self.numel();
 311:   TORCH_CHECK(
 312:       num_inp <= INT_MAX, "num_inp ", num_inp, " is too big to for CUB");
 313:   if (num_inp == 0) {
 314:     Tensor output = at::empty({0}, self.options());
 315:     Tensor inverse_indices = at::empty(self.sizes(), self.options().dtype(kLong));
 316:     Tensor counts = at::empty({0}, self.options().dtype(kLong));
 317:     return std::tuple<Tensor, Tensor, Tensor>(output, inverse_indices, counts);
 318:   }
 319: 
 320:   auto self_c = self.expect_contiguous();
 321:   return UniqueCub<scalar_t>{}(*self_c, consecutive, return_inverse, return_counts);
 322: }
```
- EN: This block defines or continues the implementation of `unique_cuda_template`.
- CN: 该代码块定义或继续实现 `unique_cuda_template`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 324-324
```cpp
 324: #define INSTANTIATE_UNIQUE_CUDA_TEMPLATE(TYPE)                            \
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 325-329
```cpp
 325:   template std::tuple<Tensor, Tensor, Tensor> unique_cuda_template<TYPE>( \
 326:       const Tensor& self,                                                 \
 327:       const bool consecutive,                                             \
 328:       const bool return_inverse,                                          \
 329:       const bool return_counts)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 331-343
```cpp
 331: INSTANTIATE_UNIQUE_CUDA_TEMPLATE(uint8_t);
 332: INSTANTIATE_UNIQUE_CUDA_TEMPLATE(int8_t);
 333: INSTANTIATE_UNIQUE_CUDA_TEMPLATE(double);
 334: INSTANTIATE_UNIQUE_CUDA_TEMPLATE(float);
 335: INSTANTIATE_UNIQUE_CUDA_TEMPLATE(int32_t);
 336: INSTANTIATE_UNIQUE_CUDA_TEMPLATE(int64_t);
 337: INSTANTIATE_UNIQUE_CUDA_TEMPLATE(int16_t);
 338: INSTANTIATE_UNIQUE_CUDA_TEMPLATE(uint32_t);
 339: INSTANTIATE_UNIQUE_CUDA_TEMPLATE(uint64_t);
 340: INSTANTIATE_UNIQUE_CUDA_TEMPLATE(uint16_t);
 341: INSTANTIATE_UNIQUE_CUDA_TEMPLATE(bool);
 342: INSTANTIATE_UNIQUE_CUDA_TEMPLATE(BFloat16);
 343: INSTANTIATE_UNIQUE_CUDA_TEMPLATE(at::Half);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 345-345
```cpp
 345: #undef INSTANTIATE
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 347-347
```cpp
 347: } // namespace at::native::internal
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `CUDA_KERNEL_LOOP` expands index-space iteration on the GPU. / `CUDA_KERNEL_LOOP` 展开 GPU 上的索引空间循环。
- CUB primitives provide parallel CUDA building blocks such as reductions or scans. / CUB 原语提供归约、扫描等并行 CUDA 基础组件。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/native/cuda/UniqueCub.cuh>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/detail/KernelUtils.h>`
  - `<ATen/cuda/CUDAApplyUtils.cuh>`
  - `<ATen/cuda/cub.cuh>`
  - `<c10/core/DeviceArray.h>`
  - `<c10/util/Load.h>`
  - `<ATen/Functions.h>`
  - `<ATen/ops/arange.h>`
  - `<ATen/ops/empty.h>`
- Runtime symbols / 运行时符号:
  - `cub::inclusive_sum_truncating`
  - `cub::unique`
  - `cub::run_length_encode`
  - `cub::radix_sort_keys`
  - `cub::radix_sort_pairs`
  - `cub::reduce`
  - `at::cuda::getCurrentCUDAStream`
  - `at::cuda::cub::inclusive_sum_truncating`
  - `at::cuda::cub::run_length_encode`
  - `at::cuda::detail::CUDA_NUM_THREADS`
  - `at::cuda::getCUDADeviceAllocator`
  - `at::cuda::cub::reduce`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
