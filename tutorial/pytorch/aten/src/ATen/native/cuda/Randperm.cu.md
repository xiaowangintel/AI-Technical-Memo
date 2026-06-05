# Randperm.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/Randperm.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `randperm_out_cuda`.
- 用途（中文）: 实现与 `randperm_out_cuda` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/Dispatch.h>
   4: #include <ATen/cuda/CUDAContext.h>
   5: #include <ATen/native/TensorFactories.h>
   6: #include <ATen/cuda/cub.h>
   7: #include <ATen/native/cuda/Randperm.cuh>
   8: 
   9: #ifndef AT_PER_OPERATOR_HEADERS
  10: #include <ATen/Functions.h>
  11: #include <ATen/NativeFunctions.h>
  12: #else
  13: #include <ATen/ops/arange.h>
  14: #include <ATen/ops/empty.h>
  15: #include <ATen/ops/empty_like.h>
  16: #include <ATen/ops/randperm_native.h>
  17: #endif
  18: 
  19: #include <limits>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/cuda/CUDAContext.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/cuda/CUDAContext.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 21-42
```cpp
  21: namespace at::native {
  22: 
  23: // [Algorithm of randperm]
  24: //
  25: // randperm is implemented by sorting an arange tensor of size n with randomly
  26: // generated keys. When random keys are different from each other, all different
  27: // permutations have the same probability.
  28: //
  29: // However, there is a pitfall here:
  30: // For better performance, these N random keys are generated independently,
  31: // and there is no effort to make sure they are different at the time of generation.
  32: // When two keys are identical, stable sorting algorithms will not permute these two keys.
  33: // As a result, (0, 1) will appear more often than (1, 0).
  34: //
  35: // To overcome this pitfall we first carefully choose the number of bits in these keys,
  36: // so that the probability of having duplicate keys is under a threshold. Let q be the
  37: // threshold probability for having non-duplicate keys, then it can be proved that[1]
  38: // the number of bits required is: ceil(log2(n - (6 n^2 + 1) / (12 log(q))))
  39: //
  40: // Then after sort, we launch a separate kernel that additionally shuffles any islands
  41: // of values whose keys matched. The algorithm of this kernel is as follows:
  42: // Each thread reads its key and the keys of its neighbors to tell if it's part of an island.
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 43-53
```cpp
  43: // For each island, the first thread in the island sees a key match at index i+1 but not index i-1.
  44: // This thread considers itself the "island leader". The island leader then reads more indices to
  45: // the right to figure out how big the island is. Most likely, the island will be very small,
  46: // just a few values. The island leader then rolls that many RNG, uses them to additionally
  47: // shuffle values within the island using serial Fisher-Yates, and writes them out.
  48: //
  49: // Reference
  50: // [1] https://osf.io/af2hy/
  51: 
  52: // The kernels are templated on an opaque, self-aligned type of the correct
  53: // size to avoid redundant kernels for different types of the same size.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 54-56
```cpp
  54: namespace {
  55: template <int N> struct alignas(N) OpaqueType { char data[N]; };
  56: }
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 58-78
```cpp
  58: Tensor& randperm_out_cuda(int64_t n, std::optional<Generator> generator, Tensor& result) {
  59:   TORCH_CHECK(n >= 0, "n must be non-negative, got", n);
  60: 
  61:   check_supported_max_int_with_precision(n, result);
  62: 
  63:   result.resize_({n});
  64: 
  65:   auto range = at::arange(n, result.options());
  66: 
  67:   // shuffled_data points to the underlying data of the output tensor if the tensor is contiguous; otherwise it
  68:   // points to a new tensor.
  69:   Tensor shuffled;
  70:   void *shuffled_data;
  71:   if (result.is_contiguous()) {
  72:     shuffled_data = result.data_ptr();
  73:   } else {
  74:     shuffled = at::empty(n, result.options());
  75:     shuffled_data = shuffled.data_ptr();
  76:   }
  77: 
  78:   auto opt = TensorOptions().device(result.device());
```
- EN: This block defines or continues the implementation of `randperm_out_cuda`.
- CN: 该代码块定义或继续实现 `randperm_out_cuda`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 80-80
```cpp
  80:   // See note [Algorithm of randperm]
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 81-82
```cpp
  81:   const double log_threshold_12 = std::log(0.9) * 12;
  82:   double nd = static_cast<double>(n);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 84-85
```cpp
  84:   int bits = std::min(64,
  85:     static_cast<int>(std::ceil(std::log2(nd - (6 * nd * nd + 1) / log_threshold_12))));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 87-108
```cpp
  87:   if (n == 0) {
  88:     return result;
  89:   } else if (bits <= 32) {
  90:     // For asserting device type match of the generator and result,
  91:     // we deligate that to the 'random_' function below.
  92: 
  93:     auto keys = at::empty(result.sizes(), opt.dtype(kInt)).random_(
  94:       std::numeric_limits<int>::min(), std::numeric_limits<int>::max(), generator);
  95:     auto keys_tmp = at::empty_like(keys);
  96:     auto keys_out = keys_tmp.mutable_data_ptr<int>();
  97:     AT_DISPATCH_ALL_TYPES_AND(kHalf, result.scalar_type(), "randperm_out_cuda", [&] {
  98:       using dtype = OpaqueType<sizeof(scalar_t)>;
  99:       auto shuffled_data_ = reinterpret_cast<dtype*>(shuffled_data);
 100:       auto* range_data = reinterpret_cast<const dtype*>(range.const_data_ptr());
 101:       at::cuda::cub::radix_sort_pairs<int, dtype>(
 102:         keys.const_data_ptr<int>(), keys_out,
 103:         range_data, shuffled_data_,
 104:         n, false, 0, bits);
 105: 
 106:       randperm_handle_duplicate_keys(keys_out, shuffled_data_, bits, n, generator);
 107:     });
 108:   } else {
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 109-123
```cpp
 109:     auto keys = at::empty(result.sizes(), opt.dtype(kLong)).random_(
 110:       std::numeric_limits<int64_t>::min(), std::numeric_limits<int64_t>::max(), generator);
 111:     auto keys_tmp = at::empty_like(keys);
 112:     auto keys_out = keys_tmp.mutable_data_ptr<int64_t>();
 113:     AT_DISPATCH_ALL_TYPES_AND(kHalf, result.scalar_type(), "randperm_out_cuda", [&] {
 114:       using dtype = OpaqueType<sizeof(scalar_t)>;
 115:       auto shuffled_data_ = reinterpret_cast<dtype*>(shuffled_data);
 116:       auto* range_data = reinterpret_cast<const dtype*>(range.data_ptr());
 117:       at::cuda::cub::radix_sort_pairs<int64_t, dtype>(
 118:         keys.const_data_ptr<int64_t>(), keys_out,
 119:         range_data, shuffled_data_,
 120:         n, false, 0, bits);
 121: 
 122:       randperm_handle_duplicate_keys(keys_out, shuffled_data_, bits, n, generator);
 123:     });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 124-124
```cpp
 124:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 126-128
```cpp
 126:   if (!result.is_contiguous()) {
 127:     result.copy_(shuffled);
 128:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 130-131
```cpp
 130:   return result;
 131: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 133-133
```cpp
 133: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- CUB primitives provide parallel CUDA building blocks such as reductions or scans. / CUB 原语提供归约、扫描等并行 CUDA 基础组件。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/native/TensorFactories.h>`
  - `<ATen/cuda/cub.h>`
  - `<ATen/native/cuda/Randperm.cuh>`
  - `<ATen/Functions.h>`
  - `<ATen/NativeFunctions.h>`
  - `<ATen/ops/arange.h>`
  - `<ATen/ops/empty.h>`
  - `<ATen/ops/empty_like.h>`
  - `<ATen/ops/randperm_native.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_ALL_TYPES_AND`
  - `cub::radix_sort_pairs`
  - `at::cuda::cub::radix_sort_pairs`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
