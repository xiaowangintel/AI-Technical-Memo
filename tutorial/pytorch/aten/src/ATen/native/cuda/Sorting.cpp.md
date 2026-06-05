# Sorting.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/Sorting.cpp`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Provides host-side CUDA entry points, orchestration, or dispatch glue around `kthvalue_out_impl_cuda`, `median_with_indices_impl`, `median_impl`, `kthvalue_out_cuda`.
- 用途（中文）: 提供围绕 `kthvalue_out_impl_cuda`, `median_with_indices_impl`, `median_impl`, `kthvalue_out_cuda` 的主机端 CUDA 入口、调度编排或分发胶水代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/native/cuda/Sorting.h>
   3: #include <ATen/core/Tensor.h>
   4: #include <ATen/core/NamedTensor.h>
   5: #include <ATen/Context.h>
   6: #include <ATen/TensorUtils.h>
   7: #include <ATen/MemoryOverlap.h>
   8: #include <ATen/WrapDimUtils.h>
   9: #include <ATen/cuda/CUDAContext.h>
  10: #include <ATen/cuda/detail/TensorInfo.cuh>
  11: 
  12: #include <ATen/native/SortingUtils.h>
  13: #include <ATen/native/ReduceOpsUtils.h>
  14: 
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/Sorting.h>`, `<ATen/core/Tensor.h>`, `<ATen/core/NamedTensor.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/Sorting.h>`, `<ATen/core/Tensor.h>`, `<ATen/core/NamedTensor.h>`。

### Lines 15-27
```cpp
  15: #ifndef AT_PER_OPERATOR_HEADERS
  16: #include <ATen/Functions.h>
  17: #include <ATen/NativeFunctions.h>
  18: #else
  19: #include <ATen/ops/full.h>
  20: #include <ATen/ops/kthvalue_native.h>
  21: #include <ATen/ops/median_native.h>
  22: #include <ATen/ops/nanmedian_native.h>
  23: #include <ATen/ops/where.h>
  24: #include <ATen/ops/rsub.h>
  25: #include <ATen/ops/div.h>
  26: #include <ATen/ops/index.h>
  27: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/full.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/full.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 29-50
```cpp
  29: namespace at::native {
  30: namespace {
  31: 
  32: std::tuple<Tensor&, Tensor&> kthvalue_out_impl_cuda(
  33:     Tensor& values,
  34:     Tensor& indices,
  35:     const Tensor& self,
  36:     int64_t k,
  37:     int64_t dim_,
  38:     bool keepdim) {
  39:   int64_t dim = maybe_wrap_dim(dim_, self.dim());
  40:   int64_t slicesize = self.dim() == 0 ? 1 : self.size(dim);
  41:   zero_numel_check_dims(self, dim, "kthvalue()");
  42: 
  43:   TORCH_CHECK(k >= 1 && k <= slicesize,
  44:               "kthvalue(): selected number k out of range for dimension ", dim);
  45: 
  46:   TORCH_CHECK(
  47:       slicesize <= std::numeric_limits<int32_t>::max(),
  48:       "kthvalue(): dimension ", dim, " is too large (", slicesize,
  49:       "). The current CUDA implementation supports dimension sizes up to ",
  50:       std::numeric_limits<int32_t>::max());
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `kthvalue_out_impl_cuda`.
- CN: 该代码块定义或继续实现 `kthvalue_out_impl_cuda`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 52-52
```cpp
  52:   at::assert_no_overlap(self, values);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 54-60
```cpp
  54:   _reduction_with_indices_allocate_or_resize_output(
  55:       values, indices, self, dim, keepdim);
  56:   if (self.dim() == 0 && self.numel() == 1) {
  57:     values.copy_(self);
  58:     indices.zero_();
  59:     return std::forward_as_tuple(values, indices);
  60:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 62-66
```cpp
  62:   TORCH_CHECK(
  63:       self.dim() <= MAX_TENSORINFO_DIMS,
  64:       "cannot operate on more than ",
  65:       MAX_TENSORINFO_DIMS,
  66:       " dimensions");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 68-69
```cpp
  68:   // Based on required index size, run the algorithm with the
  69:   // appropriate index type
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 70-72
```cpp
  70:   if (self.numel() != 0) {
  71:     launch_kthvalue_kernel(values, indices, self, dim, k);
  72:   }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 74-79
```cpp
  74:   if (!keepdim) {
  75:     values.squeeze_(dim);
  76:     indices.squeeze_(dim);
  77:   }
  78:   return std::forward_as_tuple(values, indices);
  79: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 81-102
```cpp
  81: std::tuple<Tensor&, Tensor&> median_with_indices_impl(
  82:     Tensor& values,
  83:     Tensor& indices,
  84:     const Tensor& self,
  85:     int64_t dim,
  86:     bool keepdim,
  87:     bool ignore_nan) {
  88:   // See note [Writing Nondeterministic Operations]
  89:   // If there are duplicate elements of a median value, the procedure for choosing which
  90:   // of the duplicates to use for the indices output is nondeterministic.
  91:   at::globalContext().alertNotDeterministic("median CUDA with indices output");
  92:   NoNamesGuard guard;
  93: 
  94:   dim = at::maybe_wrap_dim(dim, self.dim());
  95:   Tensor in = self.dim() > 0 ? self.contiguous() : self.unsqueeze(0);
  96: 
  97:   checkDeviceType("median", {values, indices}, self.device().type());
  98:   checkScalarType("median", {indices, "indices", 1}, kLong);
  99:   checkSameType("median", {values, "values", 0}, {self, "self", 2});
 100: 
 101:   TORCH_CHECK(
 102:       self.dim() <= MAX_TENSORINFO_DIMS,
```
- EN: This block defines or continues the implementation of `median_with_indices_impl`.
- CN: 该代码块定义或继续实现 `median_with_indices_impl`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 103-105
```cpp
 103:       "median() cannot operate on more than ",
 104:       MAX_TENSORINFO_DIMS,
 105:       " dimensions");
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 107-118
```cpp
 107:   std::vector<int64_t> out_shape = self.sizes().vec();
 108:   zero_numel_check_dims(self, dim, "median()");
 109:   if (self.dim() > 0) {
 110:     assert(dim >= 0);
 111:     assert(dim < static_cast<int64_t>(out_shape.size()));
 112: 
 113:     if (keepdim) {
 114:       out_shape[dim] = 1;
 115:     } else {
 116:       out_shape.erase(out_shape.begin() + dim);
 117:     }
 118:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 120-121
```cpp
 120:   values.resize_(out_shape);
 121:   indices.resize_(out_shape);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 123-123
```cpp
 123:   // Only launch kernel for non-empty tensors
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 124-130
```cpp
 124:   if (self.numel() > 0) {
 125:     // Ensure #dim is the same for all tensors required for reduction
 126:     Tensor vals = keepdim && self.dim() > 0 ? values : values.unsqueeze(dim);
 127:     Tensor inds = keepdim && self.dim() > 0 ? indices : indices.unsqueeze(dim);
 128: 
 129:     launch_median_kernel(vals, inds, in, dim, ignore_nan);
 130:   }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 132-134
```cpp
 132:   guard.reset();
 133:   namedinference::propagate_names_for_reduction(values, self, dim, keepdim);
 134:   namedinference::propagate_names_for_reduction(indices, self, dim, keepdim);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 136-137
```cpp
 136:   return std::forward_as_tuple(values, indices);
 137: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 139-160
```cpp
 139: Tensor median_impl(const Tensor& self, bool ignore_nan) {
 140:   NoNamesGuard guard;
 141: 
 142:   int64_t size = self.numel();
 143:   // Return nan for empty tensors
 144:   if (size <= 0) {
 145:     return at::full({}, std::numeric_limits<float>::quiet_NaN()).to(self.options());
 146:   }
 147: 
 148:   // Sort input tensor to efficiently query for median element
 149:   Tensor sorted = std::get<0>(self.flatten().sort());
 150: 
 151:   if (!ignore_nan) {
 152:     // For torch.median return either the middle element or nan (sorted as
 153:     // largest) if there are any
 154:     int64_t k = (size - 1) / 2;
 155:     return at::where(sorted[-1].isnan(), sorted[-1], sorted[k]);
 156:   } else {
 157:     // For torch.nanmedian return the middle element among the non-nan values
 158:     Tensor k = at::div(at::rsub(sorted.isnan().sum(), (size - 1)), 2).to(kLong);
 159:     return at::index(sorted, {k});
 160:   }
```
- EN: This block defines or continues the implementation of `median_impl`.
- CN: 该代码块定义或继续实现 `median_impl`。

### Lines 161-161
```cpp
 161: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 163-163
```cpp
 163: } // namespace (anonymous)
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 165-180
```cpp
 165: std::tuple<Tensor&, Tensor&> kthvalue_out_cuda(
 166:     const Tensor& self,
 167:     int64_t k,
 168:     int64_t dim,
 169:     bool keepdim,
 170:     Tensor& values,
 171:     Tensor& indices) {
 172:   auto result = [&]() {
 173:     NoNamesGuard guard;
 174:     // `kthvalue_out_impl_cuda` expects contiguous in input `self`.
 175:     return kthvalue_out_impl_cuda(values, indices, self.contiguous(), k, dim, keepdim);
 176:   }();
 177:   namedinference::propagate_names_for_reduction(values, self, dim, keepdim);
 178:   namedinference::propagate_names_for_reduction(indices, self, dim, keepdim);
 179:   return result;
 180: }
```
- EN: This block defines or continues the implementation of `kthvalue_out_cuda`.
- CN: 该代码块定义或继续实现 `kthvalue_out_cuda`。

### Lines 182-182
```cpp
 182: // Mark: median
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 184-192
```cpp
 184: std::tuple<Tensor&, Tensor&> median_out_cuda(
 185:     const Tensor& self,
 186:     int64_t dim,
 187:     bool keepdim,
 188:     Tensor& values,
 189:     Tensor& indices) {
 190:   return median_with_indices_impl(
 191:       values, indices, self, dim, keepdim, /*ignore_nan=*/false);
 192: }
```
- EN: This block defines or continues the implementation of `median_out_cuda`.
- CN: 该代码块定义或继续实现 `median_out_cuda`。

### Lines 194-196
```cpp
 194: Tensor median_cuda(const Tensor& self) {
 195:   return median_impl(self, /*ignore_nan=*/false);
 196: }
```
- EN: This block defines or continues the implementation of `median_cuda`.
- CN: 该代码块定义或继续实现 `median_cuda`。

### Lines 198-206
```cpp
 198: std::tuple<Tensor&, Tensor&> nanmedian_out_cuda(
 199:     const Tensor& self,
 200:     int64_t dim,
 201:     bool keepdim,
 202:     Tensor& values,
 203:     Tensor& indices) {
 204:   return median_with_indices_impl(
 205:       values, indices, self, dim, keepdim, /*ignore_nan=*/true);
 206: }
```
- EN: This block defines or continues the implementation of `nanmedian_out_cuda`.
- CN: 该代码块定义或继续实现 `nanmedian_out_cuda`。

### Lines 208-210
```cpp
 208: Tensor nanmedian_cuda(const Tensor& self) {
 209:   return median_impl(self, /*ignore_nan=*/true);
 210: }
```
- EN: This block defines or continues the implementation of `nanmedian_cuda`.
- CN: 该代码块定义或继续实现 `nanmedian_cuda`。

### Lines 212-212
```cpp
 212: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/native/cuda/Sorting.h>`
  - `<ATen/core/Tensor.h>`
  - `<ATen/core/NamedTensor.h>`
  - `<ATen/Context.h>`
  - `<ATen/TensorUtils.h>`
  - `<ATen/MemoryOverlap.h>`
  - `<ATen/WrapDimUtils.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/detail/TensorInfo.cuh>`
  - `<ATen/native/SortingUtils.h>`
  - `<ATen/native/ReduceOpsUtils.h>`
  - `<ATen/Functions.h>`
- Runtime symbols / 运行时符号:
  - `launch_kthvalue_kernel`
  - `launch_median_kernel`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
