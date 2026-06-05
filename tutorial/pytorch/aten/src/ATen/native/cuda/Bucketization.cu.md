# Bucketization.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/Bucketization.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `lower_bound`, `upper_bound`, `searchsorted_cuda_kernel`, `searchsorted_cuda_contiguous`.
- 用途（中文）: 实现与 `lower_bound`, `upper_bound`, `searchsorted_cuda_kernel`, `searchsorted_cuda_contiguous` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/ceil_div.h>
   4: #include <ATen/Dispatch.h>
   5: #include <ATen/cuda/CUDAContext.h>
   6: #include <ATen/native/BucketizationUtils.h>
   7: #include <ATen/native/Resize.h>
   8: 
   9: #ifndef AT_PER_OPERATOR_HEADERS
  10: #include <ATen/Functions.h>
  11: #include <ATen/NativeFunctions.h>
  12: #else
  13: #include <ATen/ops/bucketize_native.h>
  14: #include <ATen/ops/empty.h>
  15: #include <ATen/ops/searchsorted_native.h>
  16: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/ceil_div.h>`, `<ATen/Dispatch.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/ceil_div.h>`, `<ATen/Dispatch.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 18-39
```cpp
  18: namespace at::native {
  19: 
  20: // Implement a numpy like searchsorted and a TF like bucketize function running on cuda
  21: // See details in ATen/native/Bucketization.cpp
  22: 
  23: namespace {
  24: 
  25: template<typename input_t>
  26: __device__ int64_t lower_bound(const input_t *data_ss, int64_t start, int64_t end, const input_t val, const int64_t *data_sort) {
  27:   // sorter gives relative ordering for ND tensors, so we need to save and add the non-updated start as an offset
  28:   // i.e. the second row of a 3x3 tensors starts at element 3 but sorter's second row only contains 0, 1, or 2
  29:   const int64_t orig_start = start;
  30:   while (start < end) {
  31:     const int64_t mid = start + ((end - start) >> 1);
  32:     const input_t mid_val = data_sort ? data_ss[orig_start + data_sort[mid]] : data_ss[mid];
  33:     if (!(mid_val >= val)) {
  34:       start = mid + 1;
  35:     }
  36:     else {
  37:       end = mid;
  38:     }
  39:   }
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `lower_bound`.
- CN: 该代码块定义或继续实现 `lower_bound`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 40-41
```cpp
  40:   return start;
  41: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 43-59
```cpp
  43: template<typename input_t>
  44: __device__ int64_t upper_bound(const input_t *data_ss, int64_t start, int64_t end, const input_t val, const int64_t *data_sort) {
  45:   // sorter gives relative ordering for ND tensors, so we need to save and add the non-updated start as an offset
  46:   // i.e. the second row of a 3x3 tensors starts at element 3 but sorter's second row only contains 0, 1, or 2
  47:   const int64_t orig_start = start;
  48:   while (start < end) {
  49:     const int64_t mid = start + ((end - start) >> 1);
  50:     const input_t mid_val = data_sort ? data_ss[orig_start + data_sort[mid]] : data_ss[mid];
  51:     if (!(mid_val > val)) {
  52:       start = mid + 1;
  53:     }
  54:     else {
  55:       end = mid;
  56:     }
  57:   }
  58:   return start;
  59: }
```
- EN: This block defines or continues the implementation of `upper_bound`.
- CN: 该代码块定义或继续实现 `upper_bound`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 61-82
```cpp
  61: template<typename input_t, typename output_t>
  62: __global__ void searchsorted_cuda_kernel(
  63:   output_t *data_out,
  64:   const input_t *data_in,
  65:   const input_t *data_bd,
  66:   const int64_t *data_sort,
  67:   int64_t idim_in,
  68:   int64_t idim_bd,
  69:   int64_t numel_in,
  70:   bool right,
  71:   bool is_1d_boundaries) {
  72: 
  73:   for (int64_t tid = ((int64_t) blockIdx.x) * blockDim.x + threadIdx.x; tid < numel_in; tid += blockDim.x * gridDim.x) {
  74:     // If boundaries tensor is 1d, we always search the entire boundary tensor
  75:     int64_t start_bd = is_1d_boundaries ? 0 : tid / idim_in * idim_bd;
  76:     int64_t end_bd = start_bd + idim_bd;
  77: 
  78:     int64_t pos = !right ?
  79:       lower_bound<input_t>(data_bd, start_bd, end_bd, data_in[tid], data_sort) - start_bd :
  80:       upper_bound<input_t>(data_bd, start_bd, end_bd, data_in[tid], data_sort) - start_bd;
  81: 
  82:     // type conversion might happen here
```
- EN: This block defines GPU kernel entry point(s) `searchsorted_cuda_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `searchsorted_cuda_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 83-85
```cpp
  83:     data_out[tid] = pos;
  84:   }
  85: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 87-108
```cpp
  87: template<typename input_t, typename output_t>
  88: void searchsorted_cuda_contiguous(Tensor& result, const Tensor& input, const Tensor& boundaries, const bool& right, const Tensor& sorter) {
  89:   int64_t numel_in = input.numel();
  90:   bool is_scalar_input = input.dim() == 0 && numel_in == 1;
  91:   // inner most dim size of input and boundaries
  92:   int64_t idim_in = is_scalar_input ? 1 : input.sizes().back();
  93:   int64_t idim_bd = boundaries.sizes().back();
  94: 
  95:   const input_t *data_in = input.const_data_ptr<input_t>();
  96:   const input_t *data_bd = boundaries.const_data_ptr<input_t>();
  97:   const int64_t *data_sort = sorter.defined() ? sorter.const_data_ptr<int64_t>() : nullptr;
  98:   output_t *data_out = result.mutable_data_ptr<output_t>();
  99: 
 100:   int64_t maxThread = at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock;
 101:   int64_t maxGrid = 1024;
 102:   dim3 block = dim3(std::min(maxThread, numel_in));
 103:   dim3 grid  = dim3(std::min(maxGrid, ceil_div<int64_t>(numel_in, block.x)));
 104:   at::cuda::CUDAStream stream = at::cuda::getCurrentCUDAStream();
 105: 
 106:   searchsorted_cuda_kernel<<<grid, block, 0, stream>>>(
 107:     data_out, data_in, data_bd, data_sort, idim_in, idim_bd, numel_in, right, boundaries.dim() == 1);
 108:   C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This block defines or continues the implementation of `searchsorted_cuda_contiguous`.
- CN: 该代码块定义或继续实现 `searchsorted_cuda_contiguous`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 109-109
```cpp
 109: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 111-128
```cpp
 111: void dispatch(
 112:     Tensor& result,
 113:     const Tensor& input,
 114:     const Tensor& boundaries,
 115:     bool out_int32,
 116:     bool right,
 117:     const Tensor& sorter) {
 118:   if (!out_int32) {
 119:     AT_DISPATCH_ALL_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, input.scalar_type(), "searchsorted_out_cuda", [&] {
 120:       searchsorted_cuda_contiguous<scalar_t, int64_t>(result, input, boundaries, right, sorter);
 121:     });
 122:   }
 123:   else {
 124:     AT_DISPATCH_ALL_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, input.scalar_type(), "searchsorted_out_cuda", [&] {
 125:       searchsorted_cuda_contiguous<scalar_t, int>(result, input, boundaries, right, sorter);
 126:     });
 127:   }
 128: }
```
- EN: This block defines or continues the implementation of `dispatch`.
- CN: 该代码块定义或继续实现 `dispatch`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 130-130
```cpp
 130: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 132-153
```cpp
 132: Tensor& searchsorted_out_cuda(
 133:     const Tensor& sorted_sequence,
 134:     const Tensor& self,
 135:     bool out_int32,
 136:     bool right,
 137:     const std::optional<std::string_view> side_opt,
 138:     const std::optional<Tensor>& sorter_opt,
 139:     Tensor& result) {
 140:   // See [Note: hacky wrapper removal for optional tensor]
 141:   c10::MaybeOwned<Tensor> sorter_maybe_owned = at::borrow_from_optional_tensor(sorter_opt);
 142:   const Tensor& sorter = *sorter_maybe_owned;
 143:   searchsorted_pre_check(sorted_sequence, self, result, out_int32, right, side_opt, sorter);
 144:   resize_output(result, self.sizes());
 145: 
 146:   // we have two inputs to set right, pre_check checks that they aren't set to opposites
 147:   bool is_right = (side_opt && *side_opt == "right") || right;
 148:   if (self.numel() == 0) {
 149:     return result;
 150:   }
 151: 
 152:   // for non-contiguous result tensors, we write the output to a contiguous copy so we can later copy back, maintaining the original result tensor
 153:   Tensor out = result;
```
- EN: This block defines or continues the implementation of `searchsorted_out_cuda`.
- CN: 该代码块定义或继续实现 `searchsorted_out_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 154-169
```cpp
 154:   if (!result.is_contiguous()) {
 155:     out = result.contiguous();
 156:   }
 157:   if (sorted_sequence.is_contiguous() && self.is_contiguous() && sorted_sequence.dtype() == self.dtype() && sorter.is_contiguous()) {
 158:    dispatch(out, self, sorted_sequence, out_int32, is_right, sorter);
 159:   }
 160:   else {
 161:     Tensor trimmed_input;
 162:     Tensor trimmed_boundaries;
 163:     Tensor trimmed_sorter;
 164:     searchsorted_maybe_trim_input_tensors(trimmed_input, trimmed_boundaries, trimmed_sorter, self, sorted_sequence, sorter);
 165:     const Tensor& final_input = trimmed_input.defined() ? trimmed_input : self;
 166:     const Tensor& final_boundaries = trimmed_boundaries.defined() ? trimmed_boundaries : sorted_sequence;
 167:     const Tensor& final_sorter = trimmed_sorter.defined() ? trimmed_sorter : sorter;
 168:     dispatch(out, final_input, final_boundaries, out_int32, is_right, final_sorter);
 169:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 171-171
```cpp
 171:   // if result is non-contiguous, we wrote the answer to a copied version, so we copy back to the original result tensor
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 172-176
```cpp
 172:   if (!result.is_contiguous()) {
 173:     result.copy_(out);
 174:   }
 175:   return result;
 176: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 178-188
```cpp
 178: Tensor& searchsorted_out_cuda(
 179:     const Tensor& sorted_sequence,
 180:     const Scalar& self,
 181:     bool out_int32,
 182:     bool right,
 183:     const std::optional<std::string_view> side_opt,
 184:     const std::optional<Tensor>& sorter_opt,
 185:     Tensor& result) {
 186:   const Tensor& scalar_tensor = searchsorted_scalar_tensor(self, sorted_sequence.device());
 187:   return searchsorted_out_cuda(sorted_sequence, scalar_tensor, out_int32, right, side_opt, sorter_opt, result);
 188: }
```
- EN: This block defines or continues the implementation of `searchsorted_out_cuda`.
- CN: 该代码块定义或继续实现 `searchsorted_out_cuda`。

### Lines 190-202
```cpp
 190: Tensor searchsorted_cuda(
 191:     const Tensor& sorted_sequence,
 192:     const Tensor& self,
 193:     bool out_int32,
 194:     bool right,
 195:     const std::optional<std::string_view> side_opt,
 196:     const std::optional<Tensor>& sorter) {
 197:   ScalarType scalar_type = out_int32 ? ScalarType::Int : ScalarType::Long;
 198:   c10::TensorOptions options = TensorOptions().device(self.options().device()).dtype(scalar_type);
 199:   Tensor result = at::empty({0}, options, MemoryFormat::Contiguous);
 200:   at::native::searchsorted_out_cuda(sorted_sequence, self, out_int32, right, side_opt, sorter, result);
 201:   return result;
 202: }
```
- EN: This block defines or continues the implementation of `searchsorted_cuda`.
- CN: 该代码块定义或继续实现 `searchsorted_cuda`。

### Lines 204-213
```cpp
 204: Tensor searchsorted_cuda(
 205:     const Tensor& sorted_sequence,
 206:     const Scalar& self,
 207:     bool out_int32,
 208:     bool right,
 209:     const std::optional<std::string_view> side_opt,
 210:     const std::optional<Tensor>& sorter) {
 211:   const Tensor& scalar_tensor = searchsorted_scalar_tensor(self, sorted_sequence.device());
 212:   return searchsorted_cuda(sorted_sequence, scalar_tensor, out_int32, right, side_opt, sorter);
 213: }
```
- EN: This block defines or continues the implementation of `searchsorted_cuda`.
- CN: 该代码块定义或继续实现 `searchsorted_cuda`。

### Lines 215-219
```cpp
 215: Tensor& bucketize_out_cuda(const Tensor& self, const Tensor& boundaries, bool out_int32, bool right, Tensor& result) {
 216:   TORCH_CHECK(boundaries.dim() == 1, "boundaries tensor must be 1 dimension, but got dim(", boundaries.dim(), ")");
 217:   at::native::searchsorted_out_cuda(boundaries, self, out_int32, right, std::nullopt, std::nullopt, result);
 218:   return result;
 219: }
```
- EN: This block defines or continues the implementation of `bucketize_out_cuda`.
- CN: 该代码块定义或继续实现 `bucketize_out_cuda`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 221-227
```cpp
 221: Tensor bucketize_cuda(const Tensor& self, const Tensor& boundaries, bool out_int32, bool right) {
 222:   ScalarType scalar_type = out_int32 ? ScalarType::Int : ScalarType::Long;
 223:   c10::TensorOptions options = TensorOptions().device(self.options().device()).dtype(scalar_type);
 224:   Tensor result = at::empty({0}, options, MemoryFormat::Contiguous);
 225:   at::native::bucketize_out_cuda(self, boundaries, out_int32, right, result);
 226:   return result;
 227: }
```
- EN: This block defines or continues the implementation of `bucketize_cuda`.
- CN: 该代码块定义或继续实现 `bucketize_cuda`。

### Lines 229-231
```cpp
 229: Tensor bucketize_cuda(const Scalar& self, const Tensor& boundaries, bool out_int32, bool right) {
 230:   return bucketize_cuda(searchsorted_scalar_tensor(self, boundaries.device()), boundaries, out_int32, right);
 231: }
```
- EN: This block defines or continues the implementation of `bucketize_cuda`.
- CN: 该代码块定义或继续实现 `bucketize_cuda`。

### Lines 233-233
```cpp
 233: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/ceil_div.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/native/BucketizationUtils.h>`
  - `<ATen/native/Resize.h>`
  - `<ATen/Functions.h>`
  - `<ATen/NativeFunctions.h>`
  - `<ATen/ops/bucketize_native.h>`
  - `<ATen/ops/empty.h>`
  - `<ATen/ops/searchsorted_native.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_ALL_TYPES_AND2`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::CUDAStream`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
