# Unique.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/Unique.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `compute_unique`, `unique_dim_cuda_template`, `_unique_cuda`, `_unique2_cuda`.
- 用途（中文）: 实现与 `compute_unique`, `unique_dim_cuda_template`, `_unique_cuda`, `_unique2_cuda` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/Dispatch_v2.h>
   4: #include <ATen/cuda/CUDAContext.h>
   5: #include <ATen/cuda/ThrustAllocator.h>
   6: 
   7: #include <c10/util/Load.h>
   8: 
   9: #ifndef AT_PER_OPERATOR_HEADERS
  10: #include <ATen/Functions.h>
  11: #else
  12: #include <ATen/ops/_unique2_native.h>
  13: #include <ATen/ops/_unique_native.h>
  14: #include <ATen/ops/arange.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/Dispatch_v2.h>`, `<ATen/cuda/CUDAContext.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/Dispatch_v2.h>`, `<ATen/cuda/CUDAContext.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 15-28
```cpp
  15: #include <ATen/ops/empty.h>
  16: #include <ATen/ops/unique_consecutive_native.h>
  17: #include <ATen/ops/unique_dim_consecutive_native.h>
  18: #include <ATen/ops/unique_dim_native.h>
  19: #endif
  20: 
  21: #include <tuple>
  22: #include <iterator>
  23: #include <thrust/adjacent_difference.h>
  24: #include <thrust/execution_policy.h>
  25: #include <thrust/unique.h>
  26: #include <thrust/sort.h>
  27: #include <thrust/scan.h>
  28: #include <thrust/scatter.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/ops/empty.h>`, `<ATen/ops/unique_consecutive_native.h>`, `<ATen/ops/unique_dim_consecutive_native.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/ops/empty.h>`, `<ATen/ops/unique_consecutive_native.h>`, `<ATen/ops/unique_dim_consecutive_native.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 29-30
```cpp
  29: 
  30: #include <ATen/native/cuda/UniqueCub.cuh>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/UniqueCub.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/UniqueCub.cuh>`。

### Lines 32-53
```cpp
  32: namespace at::native {
  33: 
  34: namespace {
  35: 
  36: template <
  37:   typename policy_t, typename scalar_t,
  38:   typename equal_t, typename not_equal_t
  39: >
  40: std::tuple<Tensor, Tensor, int64_t> compute_unique(
  41:   const policy_t &policy,
  42:   scalar_t *data,
  43:   int64_t num_inp,
  44:   const Tensor &sorted_indices,
  45:   const bool return_inverse,
  46:   const bool return_counts,
  47:   TensorOptions options,
  48:   equal_t equal,
  49:   not_equal_t not_equal
  50: ) {
  51:   // inverse indices
  52:   Tensor inverse_indices;
  53:   if (!return_inverse || num_inp == 0) {
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `compute_unique`.
- CN: 该代码块定义或继续实现 `compute_unique`。

### Lines 54-67
```cpp
  54:     inverse_indices = at::empty({0}, options);
  55:   } else {
  56:     TORCH_CHECK(sorted_indices.defined(),
  57:       "return_inverse is set to true, but sorted_indices is undefined. Send a bug report!");
  58:     const int64_t *sorted_indices_ptr = sorted_indices.const_data_ptr<int64_t>();
  59:     Tensor inv_loc = at::empty({num_inp}, options);
  60:     inverse_indices = at::empty({num_inp}, options);
  61:     int64_t* inv_loc_ptr = inv_loc.mutable_data_ptr<int64_t>();
  62:     int64_t* inverse_indices_ptr = inverse_indices.mutable_data_ptr<int64_t>();
  63:     thrust::adjacent_difference(policy, data, data + num_inp, inv_loc_ptr, not_equal);
  64:     inv_loc[0] = 0;
  65:     thrust::inclusive_scan(policy, inv_loc_ptr, inv_loc_ptr + num_inp, inv_loc_ptr);
  66:     thrust::scatter(policy, inv_loc_ptr, inv_loc_ptr + num_inp, sorted_indices_ptr, inverse_indices_ptr);
  67:   }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 69-69
```cpp
  69:   // unique and count
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 70-82
```cpp
  70:   Tensor counts = at::empty({0}, options);
  71:   int64_t num_out;
  72:   if (!return_counts) {
  73:     num_out = thrust::unique(policy, data, data + num_inp, equal) - data;
  74:   } else {
  75:     Tensor range = at::arange(0, num_inp + 1, options);
  76:     int64_t *range_ptr = range.mutable_data_ptr<int64_t>();
  77:     num_out = thrust::unique_by_key(policy, data, data + num_inp, range_ptr, equal).first - data;
  78:     range[num_out] = num_inp;
  79:     counts.resize_(num_out);
  80:     int64_t* counts_ptr = counts.mutable_data_ptr<int64_t>();
  81:     thrust::adjacent_difference(policy, range_ptr + 1, range_ptr + num_out + 1, counts_ptr);
  82:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 84-86
```cpp
  84:   AT_CUDA_CHECK(cudaGetLastError());
  85:   return std::tuple<Tensor, Tensor, int64_t>(inverse_indices, counts, num_out);
  86: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 88-108
```cpp
  88: template <typename scalar_t>
  89: std::tuple<Tensor, Tensor, Tensor> unique_dim_cuda_template(
  90:   const Tensor& self,
  91:   const int64_t dim,
  92:   const bool consecutive,
  93:   const bool return_inverse,
  94:   const bool return_counts
  95: ) {
  96: 
  97:   /**
  98:     * The idea for implementing this is basically the same as unique.
  99:     * For unique_dim, we are taking the unique with respect to a index
 100:     * tensor, but during the processes, we override the compare and equal
 101:     * operator by checking the data underlying it instead. After the
 102:     * algorithm, we would use index_select to map the resulting indices
 103:     * to the result on the actual data.
 104:     */
 105: 
 106:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
 107:   at::cuda::ThrustAllocator allocator;
 108:   auto policy = thrust::cuda::par(allocator).on(stream);
```
- EN: This block defines or continues the implementation of `unique_dim_cuda_template`.
- CN: 该代码块定义或继续实现 `unique_dim_cuda_template`。

### Lines 110-112
```cpp
 110:   auto sizes = self.sizes().vec();
 111:   // check how many zero dimensions exist
 112:   auto num_zero_dims = std::count(sizes.begin(), sizes.end(), 0);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 114-114
```cpp
 114:   // tensor is not well formed as it has 0 sized dimensions
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 115-125
```cpp
 115:   if (self.size(dim) == 0){
 116:     TORCH_CHECK(
 117:         num_zero_dims == 1,
 118:         "Number of zero sized dimensions is more than one, so unique cannot be applied ")
 119:     Tensor output = at::empty(sizes, self.options());
 120:     Tensor inverse_indices =
 121:         at::empty({0}, self.options().dtype(kLong));
 122:     Tensor counts = at::empty({0}, self.options().dtype(kLong));
 123: 
 124:     return std::make_tuple(output, inverse_indices, counts);
 125:   }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 127-128
```cpp
 127:   TORCH_CHECK(num_zero_dims == 0,
 128:     "There are 0 sized dimensions, and they aren't selected, so unique cannot be applied");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 130-134
```cpp
 130:   int64_t num_inp = self.size(dim);
 131:   auto options = self.options().dtype(kLong);
 132:   Tensor input_flat = self.moveaxis(dim, 0).contiguous().view({num_inp, -1});
 133:   int64_t n = input_flat.size(1);
 134:   const scalar_t *input_flat_ptr = input_flat.const_data_ptr<scalar_t>();
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 136-153
```cpp
 136:   Tensor indices = at::arange(0, num_inp, options);
 137:   int64_t *indices_data = indices.mutable_data_ptr<int64_t>();
 138:   if (!consecutive) {
 139:     thrust::sort(policy, indices_data, indices_data + num_inp,
 140:       [=] __device__ (int64_t a, int64_t b) -> bool {
 141:         for (int64_t i = 0; i < n; ++i) {
 142:           scalar_t lhs = c10::load(&input_flat_ptr[i + a * n]);
 143:           scalar_t rhs = c10::load(&input_flat_ptr[i + b * n]);
 144:           if (lhs < rhs) {
 145:             return true;
 146:           } else if (lhs > rhs) {
 147:             return false;
 148:           }
 149:         }
 150:         return false;
 151:       }
 152:     );
 153:   }
```
- EN: This block defines or continues the implementation of `sort`.
- CN: 该代码块定义或继续实现 `sort`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 155-176
```cpp
 155:   auto [inverse_indices, counts, num_out] = compute_unique(
 156:     policy, indices_data, num_inp, indices,
 157:     return_inverse, return_counts, options,
 158:     [=] __device__ (int64_t a, int64_t b) -> bool {
 159:       for (int64_t i = 0; i < n; ++i) {
 160:         scalar_t lhs = c10::load(&input_flat_ptr[i + a * n]);
 161:         scalar_t rhs = c10::load(&input_flat_ptr[i + b * n]);
 162:         if (lhs != rhs) {
 163:           return false;
 164:         }
 165:       }
 166:       return true;
 167:     },
 168:     [=] __device__ (int64_t a, int64_t b) -> int64_t {
 169:       for (int64_t i = 0; i < n; ++i) {
 170:         scalar_t lhs = c10::load(&input_flat_ptr[i + a * n]);
 171:         scalar_t rhs = c10::load(&input_flat_ptr[i + b * n]);
 172:         if (lhs != rhs) {
 173:           return 1;
 174:         }
 175:       }
 176:       return 0;
```
- EN: This block defines or continues the implementation of `compute_unique`, `__device__`.
- CN: 该代码块定义或继续实现 `compute_unique`, `__device__`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 177-179
```cpp
 177:     }
 178:   );
 179:   indices.resize_(num_out);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 181-182
```cpp
 181:   return std::tuple<Tensor, Tensor, Tensor>(self.index_select(dim, indices), inverse_indices, counts);
 182: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 184-184
```cpp
 184: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 187-195
```cpp
 187: std::tuple<Tensor, Tensor>
 188: _unique_cuda(const Tensor& self, const bool sorted, const bool return_inverse) {
 189:   return AT_DISPATCH_V2(self.scalar_type(), "unique", AT_WRAP([&] {
 190:     // The current CUDA implementation of unique always sort due to the
 191:     // lack of hashtable implementation in thrust
 192:     auto [output, inverse, _] = internal::unique_cuda_template<scalar_t>(self, false, return_inverse, false);
 193:     return std::make_tuple(output, inverse);
 194:   }), AT_EXPAND(AT_ALL_TYPES), kBool, kBFloat16, kHalf, AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES));
 195: }
```
- EN: This block defines or continues the implementation of `_unique_cuda`.
- CN: 该代码块定义或继续实现 `_unique_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 197-204
```cpp
 197: std::tuple<Tensor, Tensor, Tensor>
 198: _unique2_cuda(const Tensor& self, const bool sorted, const bool return_inverse, const bool return_counts) {
 199:   return AT_DISPATCH_V2(self.scalar_type(), "unique", AT_WRAP([&] {
 200:     // The current CUDA implementation of unique always sort due to the
 201:     // lack of hashtable implementation in thrust
 202:     return internal::unique_cuda_template<scalar_t>(self, false, return_inverse, return_counts);
 203:   }), AT_EXPAND(AT_ALL_TYPES), kBool, kBFloat16, kHalf, AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES));
 204: }
```
- EN: This block defines or continues the implementation of `_unique2_cuda`.
- CN: 该代码块定义或继续实现 `_unique2_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 206-211
```cpp
 206: std::tuple<Tensor, Tensor, Tensor>
 207: unique_dim_cuda(const Tensor& self, const int64_t dim, const bool sorted, const bool return_inverse, const bool return_counts) {
 208:   return AT_DISPATCH_V2(self.scalar_type(), "unique_dim", AT_WRAP([&] {
 209:     return unique_dim_cuda_template<scalar_t>(self, dim, false, return_inverse, return_counts);
 210:   }), AT_EXPAND(AT_ALL_TYPES), kBool, kBFloat16, kHalf, AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES));
 211: }
```
- EN: This block defines or continues the implementation of `unique_dim_cuda`.
- CN: 该代码块定义或继续实现 `unique_dim_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 213-218
```cpp
 213: std::tuple<Tensor, Tensor, Tensor>
 214: unique_dim_consecutive_cuda(const Tensor& self, const int64_t dim, const bool return_inverse, const bool return_counts) {
 215:   return AT_DISPATCH_V2(self.scalar_type(), "unique_dim", AT_WRAP([&] {
 216:     return unique_dim_cuda_template<scalar_t>(self, dim, true, return_inverse, return_counts);
 217:   }), AT_EXPAND(AT_ALL_TYPES), kBool, kBFloat16, kHalf, AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES));
 218: }
```
- EN: This block defines or continues the implementation of `unique_dim_consecutive_cuda`.
- CN: 该代码块定义或继续实现 `unique_dim_consecutive_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 220-230
```cpp
 220: std::tuple<Tensor, Tensor, Tensor>
 221: unique_consecutive_cuda(const Tensor& self, const bool return_inverse, const bool return_counts, std::optional<int64_t> dim) {
 222:   if (!dim.has_value()) {
 223:     return AT_DISPATCH_V2(self.scalar_type(), "unique", AT_WRAP([&] {
 224:       // The current CUDA implementation of unique always sort due to the
 225:       // lack of hashtable implementation in thrust
 226:       return internal::unique_cuda_template<scalar_t>(self, true, return_inverse, return_counts);
 227:     }), AT_EXPAND(AT_ALL_TYPES), kBool, kBFloat16, kHalf, AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES));
 228:   }
 229:   return unique_dim_consecutive_cuda(self, dim.value(), return_inverse, return_counts);
 230: }
```
- EN: This block defines or continues the implementation of `unique_consecutive_cuda`.
- CN: 该代码块定义或继续实现 `unique_consecutive_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 232-232
```cpp
 232: }  // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- Thrust utilities supply STL-like CUDA algorithms and containers. / Thrust 工具提供类似 STL 的 CUDA 算法与容器。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/Dispatch_v2.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/ThrustAllocator.h>`
  - `<c10/util/Load.h>`
  - `<ATen/Functions.h>`
  - `<ATen/ops/_unique2_native.h>`
  - `<ATen/ops/_unique_native.h>`
  - `<ATen/ops/arange.h>`
  - `<ATen/ops/empty.h>`
  - `<ATen/ops/unique_consecutive_native.h>`
  - `<ATen/ops/unique_dim_consecutive_native.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_V2`
  - `thrust::adjacent_difference`
  - `thrust::inclusive_scan`
  - `thrust::scatter`
  - `thrust::unique`
  - `thrust::unique_by_key`
  - `thrust::cuda::par`
  - `thrust::sort`
  - `at::cuda::getCurrentCUDAStream`
  - `at::cuda::ThrustAllocator`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
