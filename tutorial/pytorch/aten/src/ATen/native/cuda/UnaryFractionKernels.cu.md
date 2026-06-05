# UnaryFractionKernels.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/UnaryFractionKernels.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `ceil_wrapper`, `ceil_kernel_cuda`, `gpu_kernel`, `frac_kernel_cuda`.
- 用途（中文）: 实现与 `ceil_wrapper`, `ceil_kernel_cuda`, `gpu_kernel`, `frac_kernel_cuda` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```cpp
   1: #define TORCH_ASSERT_NO_OPERATORS
   2: #include <limits>
   3: #include <ATen/native/UnaryOps.h>
   4: #include <ATen/native/cuda/Loops.cuh>
   5: #include <ATen/AccumulateType.h>
   6: #include <ATen/Dispatch.h>
   7: #include <ATen/native/DispatchStub.h>
   8: #include <ATen/native/TensorIterator.h>
   9: #include <ATen/native/cuda/Math.cuh>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<limits>`, `<ATen/native/UnaryOps.h>`, `<ATen/native/cuda/Loops.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<limits>`, `<ATen/native/UnaryOps.h>`, `<ATen/native/cuda/Loops.cuh>`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 11-32
```cpp
  11: namespace at::native {
  12: 
  13: // We manually overload ceil because std::ceil does not work with std::complex types.
  14: template <typename scalar_t>
  15: __host__ __device__ static inline scalar_t ceil_wrapper(scalar_t a) {
  16:   return std::ceil(a);
  17: }
  18: 
  19: template<typename T>
  20: __host__ __device__ static inline std::complex<T> ceil_wrapper(std::complex<T> v) {
  21:   return std::complex<T>(std::ceil(v.real()), std::ceil(v.imag()));
  22: }
  23: 
  24: void ceil_kernel_cuda(TensorIteratorBase& iter) {
  25:   AT_DISPATCH_FLOATING_TYPES_AND2(
  26:       ScalarType::Half, ScalarType::BFloat16,
  27:       iter.dtype(), "ceil_cuda",
  28:       [&]() {
  29:         gpu_kernel(iter, []GPU_LAMBDA(scalar_t a) -> scalar_t {
  30:           return ceil_wrapper(a);
  31:         });
  32:       });
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `ceil_wrapper`, `ceil_kernel_cuda`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `ceil_wrapper`, `ceil_kernel_cuda`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 33-33
```cpp
  33: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 35-44
```cpp
  35: void frac_kernel_cuda(TensorIteratorBase& iter) {
  36:   AT_DISPATCH_FLOATING_TYPES_AND2(
  37:       ScalarType::Half, ScalarType::BFloat16,
  38:       iter.dtype(), "frac_cuda",
  39:       [&]() {
  40:         gpu_kernel(iter, []GPU_LAMBDA(scalar_t a) -> scalar_t {
  41:           return a - ::trunc(a);
  42:         });
  43:       });
  44: }
```
- EN: This block defines or continues the implementation of `frac_kernel_cuda`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `frac_kernel_cuda`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 46-46
```cpp
  46: // We manually overload floor because std::floor does not work with std::complex types.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 47-50
```cpp
  47: template <typename scalar_t>
  48: __host__ __device__ static inline scalar_t floor_wrapper(scalar_t a) {
  49:   return std::floor(a);
  50: }
```
- EN: This block defines or continues the implementation of `floor_wrapper`.
- CN: 该代码块定义或继续实现 `floor_wrapper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 52-55
```cpp
  52: template<typename T>
  53: __host__ __device__ static inline std::complex<T> floor_wrapper(std::complex<T> v) {
  54:   return std::complex<T>(std::floor(v.real()), std::floor(v.imag()));
  55: }
```
- EN: This block defines or continues the implementation of `floor_wrapper`.
- CN: 该代码块定义或继续实现 `floor_wrapper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 57-66
```cpp
  57: void floor_kernel_cuda(TensorIteratorBase& iter) {
  58:   AT_DISPATCH_FLOATING_TYPES_AND2(
  59:       ScalarType::Half, ScalarType::BFloat16,
  60:       iter.dtype(), "floor_cuda",
  61:       [&]() {
  62:         gpu_kernel(iter, []GPU_LAMBDA(scalar_t a) -> scalar_t {
  63:           return floor_wrapper(a);
  64:         });
  65:       });
  66: }
```
- EN: This block defines or continues the implementation of `floor_kernel_cuda`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `floor_kernel_cuda`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 68-71
```cpp
  68: template <typename scalar_t>
  69: __host__ __device__ static inline scalar_t reciprocal_wrapper(scalar_t a) {
  70:   return static_cast<scalar_t>(1)/a;
  71: }
```
- EN: This block defines or continues the implementation of `reciprocal_wrapper`.
- CN: 该代码块定义或继续实现 `reciprocal_wrapper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 73-94
```cpp
  73: template<typename T>
  74: __host__ __device__ static inline c10::complex<T> reciprocal_wrapper(c10::complex<T> v) {
  75:   // Handle extreme cases for numpy compatibility
  76:   auto both_inf = [](T real, T imag) {
  77:     return (::isinf(real) && ::isinf(imag));
  78:   };
  79: 
  80:   auto either_inf = [](T real, T imag) {
  81:     return ::isinf(real) || ::isinf(imag);
  82:   };
  83: 
  84:   auto either_nan = [](T real, T imag) {
  85:     return ::isnan(real) || ::isnan(imag);
  86:   };
  87: 
  88:   if (either_nan(v.real(), v.imag()) || both_inf(v.real(), v.imag())) {
  89:     // If either is Nan or both are infinite, return {nan, nan}
  90:     return {std::numeric_limits<T>::quiet_NaN(), std::numeric_limits<T>::quiet_NaN()};
  91:   } else if (either_inf(v.real(), v.imag())) {
  92:     // If either is Inf, return {0, 0}
  93:     return {0, 0};
  94:   }
```
- EN: This block defines or continues the implementation of `reciprocal_wrapper`.
- CN: 该代码块定义或继续实现 `reciprocal_wrapper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 95-97
```cpp
  95:   const c10::complex<T> one = c10::complex<T>(1.0, 0);
  96:   return one/v;
  97: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 99-108
```cpp
  99: void reciprocal_kernel_cuda(TensorIteratorBase& iter) {
 100:   AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES_AND2(
 101:       ScalarType::Half, ScalarType::BFloat16,
 102:       iter.common_dtype(), "reciprocal_cuda",
 103:       [&]() {
 104:         gpu_kernel(iter, []GPU_LAMBDA(scalar_t a) -> scalar_t {
 105:           return reciprocal_wrapper(a);
 106:         });
 107:       });
 108: }
```
- EN: This block defines or continues the implementation of `reciprocal_kernel_cuda`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `reciprocal_kernel_cuda`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 110-110
```cpp
 110: // We manually overload nearbyint because std::nearbyint does not work with std::complex types and ROCm.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 111-114
```cpp
 111: template <typename scalar_t>
 112: __host__ __device__ static inline scalar_t nearbyint_wrapper(scalar_t a) {
 113:   return static_cast<scalar_t>(::nearbyintf(static_cast<float>(a)));
 114: }
```
- EN: This block defines or continues the implementation of `nearbyint_wrapper`.
- CN: 该代码块定义或继续实现 `nearbyint_wrapper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 116-118
```cpp
 116: __host__ __device__ static inline double nearbyint_wrapper(double a) {
 117:   return ::nearbyint(a);
 118: }
```
- EN: This block defines or continues the implementation of `nearbyint_wrapper`.
- CN: 该代码块定义或继续实现 `nearbyint_wrapper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 120-121
```cpp
 120: #pragma push
 121: #pragma nv_diag_suppress 177   // Function was declared but never referenced
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 122-124
```cpp
 122: __host__ __device__ static inline c10::complex<float> nearbyint_wrapper(c10::complex<float> a) {
 123:   return c10::complex<float>(::nearbyintf(static_cast<float>(a.real())), ::nearbyintf(static_cast<float>(a.imag())));
 124: }
```
- EN: This block defines or continues the implementation of `nearbyint_wrapper`.
- CN: 该代码块定义或继续实现 `nearbyint_wrapper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 126-129
```cpp
 126: __host__ __device__ static inline c10::complex<double> nearbyint_wrapper(c10::complex<double> a) {
 127:   return c10::complex<double>(::nearbyint(static_cast<double>(a.real())), ::nearbyint(static_cast<double>(a.imag())));
 128: }
 129: #pragma pop
```
- EN: This block defines or continues the implementation of `nearbyint_wrapper`.
- CN: 该代码块定义或继续实现 `nearbyint_wrapper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 131-141
```cpp
 131: void round_kernel_cuda(TensorIteratorBase& iter) {
 132:   AT_DISPATCH_FLOATING_TYPES_AND2(
 133:       ScalarType::Half, ScalarType::BFloat16,
 134:       iter.dtype(), "round_cuda",
 135:       [&]() {
 136:         gpu_kernel(iter, []GPU_LAMBDA(scalar_t a) -> scalar_t {
 137:           // We do not use std::round because we would like to round midway numbers to the nearest even integer.
 138:           return nearbyint_wrapper(a);
 139:         });
 140:       });
 141: }
```
- EN: This block defines or continues the implementation of `round_kernel_cuda`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `round_kernel_cuda`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 143-160
```cpp
 143: void round_decimals_kernel_cuda(TensorIteratorBase& iter, int64_t decimals) {
 144:   AT_DISPATCH_FLOATING_TYPES_AND2(
 145:       ScalarType::Half, ScalarType::BFloat16,
 146:       iter.dtype(), "round_cuda",
 147:       [&]() {
 148:         bool neg_flag = false;
 149:         scalar_t ten_pow_decimals;
 150:         if (decimals < 0) {
 151:           decimals = -decimals;
 152:           neg_flag = true;
 153:         }
 154:         ten_pow_decimals = static_cast<scalar_t>(std::pow(10, decimals));
 155:         gpu_kernel(iter, [ten_pow_decimals, neg_flag]GPU_LAMBDA(scalar_t a) -> scalar_t {
 156:           return neg_flag ? std::nearbyint(a / ten_pow_decimals) * ten_pow_decimals
 157:                           : std::nearbyint(a * ten_pow_decimals) / ten_pow_decimals;
 158:         });
 159:       });
 160: }
```
- EN: This block defines or continues the implementation of `round_decimals_kernel_cuda`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `round_decimals_kernel_cuda`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 162-162
```cpp
 162: // We manually overload trunc because std::trunc does not work with std::complex types and ROCm.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 163-166
```cpp
 163: template <typename scalar_t>
 164: __host__ __device__ static inline scalar_t trunc_wrapper(scalar_t a) {
 165:   return static_cast<scalar_t>(::truncf(static_cast<float>(a)));
 166: }
```
- EN: This block defines or continues the implementation of `trunc_wrapper`.
- CN: 该代码块定义或继续实现 `trunc_wrapper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 168-170
```cpp
 168: __host__ __device__ static inline double trunc_wrapper(double a) {
 169:   return ::trunc(a);
 170: }
```
- EN: This block defines or continues the implementation of `trunc_wrapper`.
- CN: 该代码块定义或继续实现 `trunc_wrapper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 172-174
```cpp
 172: __host__ __device__ static inline c10::complex<float> trunc_wrapper(c10::complex<float> a) {
 173:   return c10::complex<float>(::truncf(static_cast<float>(a.real())), ::truncf(static_cast<float>(a.imag())));
 174: }
```
- EN: This block defines or continues the implementation of `trunc_wrapper`.
- CN: 该代码块定义或继续实现 `trunc_wrapper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 176-178
```cpp
 176: __host__ __device__ static inline c10::complex<double> trunc_wrapper(c10::complex<double> a) {
 177:   return c10::complex<double>(::trunc(static_cast<double>(a.real())), ::trunc(static_cast<double>(a.imag())));
 178: }
```
- EN: This block defines or continues the implementation of `trunc_wrapper`.
- CN: 该代码块定义或继续实现 `trunc_wrapper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 180-189
```cpp
 180: void trunc_kernel_cuda(TensorIteratorBase& iter) {
 181:   AT_DISPATCH_FLOATING_TYPES_AND2(
 182:       ScalarType::Half, ScalarType::BFloat16,
 183:       iter.dtype(), "trunc_cuda",
 184:       [&]() {
 185:         gpu_kernel(iter, []GPU_LAMBDA(scalar_t a) -> scalar_t {
 186:           return trunc_wrapper(a);
 187:         });
 188:       });
 189: }
```
- EN: This block defines or continues the implementation of `trunc_kernel_cuda`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `trunc_kernel_cuda`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 191-197
```cpp
 191: REGISTER_DISPATCH(ceil_stub, &ceil_kernel_cuda)
 192: REGISTER_DISPATCH(frac_stub, &frac_kernel_cuda)
 193: REGISTER_DISPATCH(floor_stub, &floor_kernel_cuda)
 194: REGISTER_DISPATCH(reciprocal_stub, &reciprocal_kernel_cuda)
 195: REGISTER_DISPATCH(round_stub, &round_kernel_cuda)
 196: REGISTER_DISPATCH(round_decimals_stub, &round_decimals_kernel_cuda)
 197: REGISTER_DISPATCH(trunc_stub, &trunc_kernel_cuda)
```
- EN: The registration macro binds this implementation into PyTorch dispatch so higher-level operators can call the CUDA specialization.
- CN: 注册宏把该实现接入 PyTorch 分发系统，使上层算子能够调用这个 CUDA 特化版本。

### Lines 199-199
```cpp
 199: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TensorIterator` normalizes tensor shapes/strides and drives elementwise CUDA traversal. / `TensorIterator` 统一张量形状与步长，并驱动逐元素 CUDA 遍历。
- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `gpu_kernel` applies a functor over iterator elements on CUDA. / `gpu_kernel` 在 CUDA 上对迭代器元素应用函数对象。
- Dispatch registration exposes the CUDA specialization to higher PyTorch layers. / 分发注册把 CUDA 特化实现暴露给更高层的 PyTorch 调用链。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<limits>`
  - `<ATen/native/UnaryOps.h>`
  - `<ATen/native/cuda/Loops.cuh>`
  - `<ATen/AccumulateType.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/native/DispatchStub.h>`
  - `<ATen/native/TensorIterator.h>`
  - `<ATen/native/cuda/Math.cuh>`
- Runtime symbols / 运行时符号:
  - `ceil_stub`
  - `frac_stub`
  - `floor_stub`
  - `reciprocal_stub`
  - `round_stub`
  - `round_decimals_stub`
  - `trunc_stub`
  - `gpu_kernel`
  - `TensorIterator`
  - `TensorIteratorBase`
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES_AND2`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
