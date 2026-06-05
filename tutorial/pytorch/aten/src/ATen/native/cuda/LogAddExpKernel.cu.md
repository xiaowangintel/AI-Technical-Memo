# LogAddExpKernel.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/LogAddExpKernel.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `_logaddexp_minmax`, `_log_add_exp_helper`, `_fast_build_exp`, `_fast_build_exp_inf`.
- 用途（中文）: 实现与 `_logaddexp_minmax`, `_log_add_exp_helper`, `_fast_build_exp`, `_fast_build_exp_inf` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
   1: #define TORCH_ASSERT_NO_OPERATORS
   2: #include <ATen/Dispatch.h>
   3: #include <ATen/native/DispatchStub.h>
   4: #include <ATen/native/cuda/Loops.cuh>
   5: #include <ATen/native/cuda/JitLoops.cuh>
   6: #include <ATen/native/cuda/jit_utils.h>
   7: #include <ATen/native/cuda/ScanUtils.cuh>
   8: #include <ATen/native/TensorIterator.h>
   9: #include <ATen/native/BinaryOps.h>
  10: #include <ATen/OpMathType.h>
  11: #include <c10/util/MathConstants.h>
  12: #include <c10/util/complex.h>
  13: 
  14: #include <cmath>
  15: #include <limits>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Dispatch.h>`, `<ATen/native/DispatchStub.h>`, `<ATen/native/cuda/Loops.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Dispatch.h>`, `<ATen/native/DispatchStub.h>`, `<ATen/native/cuda/Loops.cuh>`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 17-18
```cpp
  17: // NOTE: CUDA on Windows requires that the enclosing function
  18: // of a __device__ lambda not have internal linkage.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 20-41
```cpp
  20: namespace at::native {
  21: 
  22: // custom min and max to be used in logaddexp for  complex arguments
  23: template <typename scalar_t, bool min>
  24: __host__ __device__ c10::complex<scalar_t> _logaddexp_minmax(const c10::complex<scalar_t>& x, const c10::complex<scalar_t>& y) {
  25:   scalar_t xr = std::real(x);
  26:   scalar_t yr = std::real(y);
  27:   if (::isnan(yr) || (::isnan(std::imag(y)))) {
  28:     return y;
  29:   } else if (::isnan(xr) || (::isnan(std::imag(x)))) {
  30:     return x;
  31:   } else if (min) { // min
  32:     return (xr < yr) ? x : y;
  33:   } else { // max
  34:     return (xr >= yr) ? x : y;
  35:   }
  36: }
  37: 
  38: template <typename scalar_t>
  39: __host__ __device__ scalar_t _log_add_exp_helper(const scalar_t& x, const scalar_t& y) {
  40:   // Reference : https://www.tensorflow.org/api_docs/python/tf/math/cumulative_logsumexp
  41:   // Using the original expression: `at::_isnan(y) ? y : std::min(x, y)` causes an error in ROCM
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `_logaddexp_minmax`, `_log_add_exp_helper`.
- CN: 该代码块定义或继续实现 `_logaddexp_minmax`, `_log_add_exp_helper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 42-53
```cpp
  42:   const auto isnan_x = at::_isnan(x);
  43:   const auto isnan_y = at::_isnan(y);
  44:   scalar_t min = isnan_y ? y : (isnan_x ? x : std::min(x, y));
  45:   scalar_t max = isnan_y ? y : (isnan_x ? x : std::max(x, y));
  46:   if (min != max || ::isfinite(min)) {
  47:     // nan will be propagated here
  48:     return ::log1p(std::exp(min - max)) + max;
  49:   } else {
  50:     // special case to correctly handle infinite cases
  51:     return x;
  52:   }
  53: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 55-65
```cpp
  55: template <typename scalar_t>
  56: __host__ __device__ c10::complex<scalar_t> _fast_build_exp(const c10::complex<scalar_t>& x) {
  57:   // complex exponential function, but implemented manually to get fast compilation time
  58:   // this function only handles the case where the x is finite (not inf nor nan)
  59:   const auto xreal = std::real(x);
  60:   const auto ximag = std::imag(x);
  61:   const auto exp_x_abs = std::exp(xreal);
  62:   auto exp_x_real = exp_x_abs * std::cos(ximag);
  63:   auto exp_x_imag = exp_x_abs * std::sin(ximag);
  64:   return {exp_x_real, exp_x_imag};
  65: }
```
- EN: This block defines or continues the implementation of `_fast_build_exp`.
- CN: 该代码块定义或继续实现 `_fast_build_exp`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 67-82
```cpp
  67: template <typename scalar_t>
  68: __host__ __device__ c10::complex<scalar_t> _fast_build_exp_inf(const c10::complex<scalar_t>& x) {
  69:   // complex exponential function, but implemented manually to get fast compilation time
  70:   // this function only handles the case where the real part of x is infinite
  71:   const auto ximag = std::imag(x);
  72:   constexpr auto exp_x_abs = std::numeric_limits<scalar_t>::infinity();
  73:   if (!::isfinite(ximag)) {  // add this to make consistent with std::exp(x+yi)
  74:     return {exp_x_abs, std::numeric_limits<scalar_t>::quiet_NaN()};
  75:   }
  76:   const auto sin = std::sin(ximag);
  77:   const auto cos = std::cos(ximag);
  78:   // special case if the angle is exactly the multiple of pi/2
  79:   auto exp_x_real = (cos == 0) ? (scalar_t)0.0 : exp_x_abs * cos;
  80:   auto exp_x_imag = (sin == 0) ? (scalar_t)0.0 : exp_x_abs * sin;
  81:   return {exp_x_real, exp_x_imag};
  82: }
```
- EN: This block defines or continues the implementation of `_fast_build_exp_inf`.
- CN: 该代码块定义或继续实现 `_fast_build_exp_inf`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 84-105
```cpp
  84: template <typename scalar_t>
  85: __host__ __device__ c10::complex<scalar_t> _log_add_exp_helper(const c10::complex<scalar_t>& x, const c10::complex<scalar_t>& y) {
  86:   c10::complex<scalar_t> min = _logaddexp_minmax<scalar_t, /*min=*/true>(x, y);
  87:   c10::complex<scalar_t> max = _logaddexp_minmax<scalar_t, /*min=*/false>(x, y);
  88:   scalar_t min_real = std::real(min);
  89:   scalar_t max_real = std::real(max);
  90: 
  91:   if (::isnan(min_real) || ::isnan(std::imag(min))) {
  92:     // handling the "infectious" NaNs
  93:     return {std::numeric_limits<scalar_t>::quiet_NaN(), std::numeric_limits<scalar_t>::quiet_NaN()};
  94:   }
  95:   else if ((!::isfinite(min_real)) && (min_real == max_real)) {
  96:     if (min_real < 0) {
  97:       // handle the -inf case, the imaginary part here does not really matter as the exp(value)
  98:       // will be around 0.0 and the angle (i.e. the imaginary part) cannot be determined.
  99:       // It does not matter if we're taking the exp of this value
 100:       return min;
 101:     } else {
 102:       // handle the +inf case, we don't need the special precision for log1p for small values
 103:       // and to avoid producing nan in case of real(max) == real(min) == +inf
 104:       const auto exp_min = _fast_build_exp_inf(min);
 105:       const auto exp_max = _fast_build_exp_inf(max);
```
- EN: This block defines or continues the implementation of `_log_add_exp_helper`.
- CN: 该代码块定义或继续实现 `_log_add_exp_helper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 106-118
```cpp
 106:       return ::log1p(exp_min + exp_max - 1);  // log1p(x - 1) builds faster than log
 107:     }
 108:   } else {
 109:     const auto minmax = min - max;
 110:     c10::complex<scalar_t> exp_minmax;
 111:     if (!::isfinite(minmax.real())) {
 112:         exp_minmax = minmax.real() < 0 ? c10::complex<scalar_t>{0.0, 0.0} : _fast_build_exp_inf(minmax);
 113:     } else {
 114:         exp_minmax = _fast_build_exp(minmax);
 115:     }
 116:     return ::log1p(exp_minmax) + max;
 117:   }
 118: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 120-120
```cpp
 120: // Complex logaddexp jiterator string
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 121-140
```cpp
 121: const auto logaddexp_complex_string = jiterator_stringify(
 122:     template<typename T>
 123:     std::complex<T> log1p(const std::complex<T>& z)
 124:     {
 125:       using complex_t = std::complex<T>;
 126:       T x = z.real();
 127:       T y = z.imag();
 128:       T zabs = abs(z);
 129:       T theta = atan2(y, x + T(1));
 130:       if (zabs < 0.5) {
 131:           T r = x * (T(2) + x) + y * y;
 132:           if (r == 0) { // handle underflow
 133:               return complex_t(x, theta);
 134:           }
 135:           return complex_t(T(0.5) * std::log1p(r), theta);
 136:       } else {
 137:           T z0 = std::hypot(x + 1, y);
 138:           return complex_t(log(z0), theta);
 139:       }
 140:     }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 142-142
```cpp
 142:     // separated _logaddexp_minmax into 2 different functions for jiterator_string
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 143-154
```cpp
 143:     template <typename T>
 144:     std::complex<T> logaddexp_min(const std::complex<T>& x, const std::complex<T>& y) {
 145:         T xr = x.real();
 146:         T yr = y.real();
 147:         if (isnan(yr) || isnan(y.imag())) {
 148:             return y;
 149:         } else if (isnan(xr) || isnan(x.imag())) {
 150:             return x;
 151:         } else {
 152:             return (xr < yr) ? x : y;
 153:         }
 154:     }
```
- EN: This block defines or continues the implementation of `logaddexp_min`.
- CN: 该代码块定义或继续实现 `logaddexp_min`。

### Lines 156-167
```cpp
 156:     template <typename T>
 157:     std::complex<T> logaddexp_max(const std::complex<T>& x, const std::complex<T>& y) {
 158:         T xr = x.real();
 159:         T yr = y.real();
 160:         if (isnan(yr) || isnan(y.imag())) {
 161:             return y;
 162:         } else if (isnan(xr) || isnan(x.imag())) {
 163:             return x;
 164:         } else {
 165:             return (xr >= yr) ? x : y;
 166:         }
 167:     }
```
- EN: This block defines or continues the implementation of `logaddexp_max`.
- CN: 该代码块定义或继续实现 `logaddexp_max`。

### Lines 169-177
```cpp
 169:     template <typename T>
 170:     std::complex<T> fast_build_exp(const std::complex<T>& x) {
 171:         const auto xreal = x.real();
 172:         const auto ximag = x.imag();
 173:         const auto exp_x_abs = exp(xreal);
 174:         auto exp_x_real = exp_x_abs * cos(ximag);
 175:         auto exp_x_imag = exp_x_abs * sin(ximag);
 176:         return std::complex<T>(exp_x_real, exp_x_imag);
 177:     }
```
- EN: This block defines or continues the implementation of `fast_build_exp`.
- CN: 该代码块定义或继续实现 `fast_build_exp`。

### Lines 179-192
```cpp
 179:     template <typename T>
 180:     std::complex<T> fast_build_exp_inf(const std::complex<T>& x) {
 181:         using complex_t = std::complex<T>;
 182:         const auto ximag = x.imag();
 183:         const T exp_x_abs = INFINITY;
 184:         if (!isfinite(ximag)) {
 185:             return complex_t(exp_x_abs, NAN);
 186:         }
 187:         const auto sin_val = sin(ximag);
 188:         const auto cos_val = cos(ximag);
 189:         auto exp_x_real = (cos_val == T(0)) ? T(0) : exp_x_abs * cos_val;
 190:         auto exp_x_imag = (sin_val == T(0)) ? T(0) : exp_x_abs * sin_val;
 191:         return complex_t(exp_x_real, exp_x_imag);
 192:     }
```
- EN: This block defines or continues the implementation of `fast_build_exp_inf`.
- CN: 该代码块定义或继续实现 `fast_build_exp_inf`。

### Lines 194-215
```cpp
 194:     template <typename complex_t>
 195:     complex_t logaddexp_complex(complex_t x, complex_t y) {
 196:         using T = typename complex_t::value_type;
 197:         complex_t min_val = logaddexp_min(x, y);
 198:         complex_t max_val = logaddexp_max(x, y);
 199:         T min_real = min_val.real();
 200:         T max_real = max_val.real();
 201: 
 202:         if (isnan(min_real) || isnan(min_val.imag())) {
 203:             return complex_t(NAN, NAN);
 204:         }
 205:         else if ((!isfinite(min_real)) && (min_real == max_real)) {
 206:             if (min_real < T(0)) {
 207:                 return min_val;
 208:             } else {
 209:                 const auto exp_min = fast_build_exp_inf<T>(min_val);
 210:                 const auto exp_max = fast_build_exp_inf<T>(max_val);
 211:                 return log1p(exp_min + exp_max - complex_t(1, 0));
 212:             }
 213:         } else {
 214:             const auto minmax = min_val - max_val;
 215:             complex_t exp_minmax;
```
- EN: This block defines or continues the implementation of `logaddexp_complex`.
- CN: 该代码块定义或继续实现 `logaddexp_complex`。

### Lines 216-224
```cpp
 216:             if (!isfinite(minmax.real())) {
 217:                 exp_minmax = (minmax.real() < T(0)) ? complex_t(0, 0) : fast_build_exp_inf<T>(minmax);
 218:             } else {
 219:                 exp_minmax = fast_build_exp<T>(minmax);
 220:             }
 221:             return log1p(exp_minmax) + max_val;
 222:         }
 223:     }
 224: );
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 226-247
```cpp
 226: constexpr char logaddexp_complex_name[] = "logaddexp_complex";
 227: void logaddexp_kernel_cuda(TensorIteratorBase& iter) {
 228:   if (at::isComplexType(iter.dtype())) {
 229: #if AT_USE_JITERATOR()
 230:     AT_DISPATCH_COMPLEX_TYPES_AND(at::ScalarType::ComplexHalf, iter.dtype(), "logaddexp_cuda", [&]() {
 231:       jitted_gpu_kernel<
 232:           /*name=*/logaddexp_complex_name,
 233:           /*return_dtype=*/scalar_t,
 234:           /*common_dtype=*/scalar_t,
 235:           /*arity=*/2>(iter, logaddexp_complex_string);
 236:     });
 237: #else
 238:     AT_DISPATCH_COMPLEX_TYPES_AND(at::ScalarType::ComplexHalf, iter.dtype(), "logaddexp_cuda", [&]() {
 239:       using opmath_t = at::opmath_type<scalar_t>;
 240:       gpu_kernel(iter, [] GPU_LAMBDA (scalar_t a_, scalar_t b_) -> scalar_t {
 241:         const auto a = static_cast<opmath_t>(a_);
 242:         const auto b = static_cast<opmath_t>(b_);
 243:         return static_cast<scalar_t>(_log_add_exp_helper(a, b));
 244:       });
 245:     });
 246: #endif
 247:   } else {
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `logaddexp_kernel_cuda`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `logaddexp_kernel_cuda`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 248-263
```cpp
 248:     AT_DISPATCH_FLOATING_TYPES_AND2(
 249:       ScalarType::BFloat16, ScalarType::Half,
 250:       iter.dtype(), "logaddexp_cuda",
 251:       [&]() {
 252:         using opmath_t = at::opmath_type<scalar_t>;
 253:         gpu_kernel(iter, [] GPU_LAMBDA (scalar_t a_, scalar_t b_) -> scalar_t {
 254:           const auto a = static_cast<opmath_t>(a_);
 255:           const auto b = static_cast<opmath_t>(b_);
 256:           if (::isinf(a) && a == b) {
 257:             return a;
 258:           } else {
 259:             const auto m = ::max(a, b);
 260:             return m + ::log1p(::exp(-::abs(a - b)));
 261:           }
 262:         });
 263:       });
```
- EN: This block defines or continues the implementation of `gpu_kernel`.
- CN: 该代码块定义或继续实现 `gpu_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 264-265
```cpp
 264:   }
 265: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 267-285
```cpp
 267: void logaddexp2_kernel_cuda(TensorIteratorBase& iter) {
 268:   AT_DISPATCH_FLOATING_TYPES_AND2(
 269:       ScalarType::BFloat16, ScalarType::Half,
 270:       iter.dtype(), "logaddexp2_cuda",
 271:       [&]() {
 272:         using opmath_t = at::opmath_type<scalar_t>;
 273:         const auto inv_log_2 = static_cast<opmath_t>(1.0 / c10::ln_2<double>);
 274:         gpu_kernel(iter, [inv_log_2] GPU_LAMBDA (scalar_t a_, scalar_t b_) -> scalar_t {
 275:           const auto a = static_cast<opmath_t>(a_);
 276:           const auto b = static_cast<opmath_t>(b_);
 277:           if (::isinf(a) && a == b) {
 278:             return a;
 279:           } else {
 280:             const auto m = ::max(a, b);
 281:             return m + ::log1p(::exp2(-::abs(a - b))) * inv_log_2;
 282:           }
 283:         });
 284:       });
 285: }
```
- EN: This block defines or continues the implementation of `logaddexp2_kernel_cuda`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `logaddexp2_kernel_cuda`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 287-288
```cpp
 287: REGISTER_DISPATCH(logaddexp_stub, &logaddexp_kernel_cuda)
 288: REGISTER_DISPATCH(logaddexp2_stub, &logaddexp2_kernel_cuda)
```
- EN: The registration macro binds this implementation into PyTorch dispatch so higher-level operators can call the CUDA specialization.
- CN: 注册宏把该实现接入 PyTorch 分发系统，使上层算子能够调用这个 CUDA 特化版本。

### Lines 290-290
```cpp
 290: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TensorIterator` normalizes tensor shapes/strides and drives elementwise CUDA traversal. / `TensorIterator` 统一张量形状与步长，并驱动逐元素 CUDA 遍历。
- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `gpu_kernel` applies a functor over iterator elements on CUDA. / `gpu_kernel` 在 CUDA 上对迭代器元素应用函数对象。
- `jitted_gpu_kernel` relies on Jiterator-style runtime code generation. / `jitted_gpu_kernel` 依赖 Jiterator 风格的运行时代码生成。
- Dispatch registration exposes the CUDA specialization to higher PyTorch layers. / 分发注册把 CUDA 特化实现暴露给更高层的 PyTorch 调用链。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/Dispatch.h>`
  - `<ATen/native/DispatchStub.h>`
  - `<ATen/native/cuda/Loops.cuh>`
  - `<ATen/native/cuda/JitLoops.cuh>`
  - `<ATen/native/cuda/jit_utils.h>`
  - `<ATen/native/cuda/ScanUtils.cuh>`
  - `<ATen/native/TensorIterator.h>`
  - `<ATen/native/BinaryOps.h>`
  - `<ATen/OpMathType.h>`
  - `<c10/util/MathConstants.h>`
  - `<c10/util/complex.h>`
  - `<cmath>`
- Runtime symbols / 运行时符号:
  - `logaddexp_stub`
  - `logaddexp2_stub`
  - `gpu_kernel`
  - `jitted_gpu_kernel`
  - `TensorIterator`
  - `TensorIteratorBase`
  - `AT_DISPATCH_COMPLEX_TYPES_AND`
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `REGISTER_DISPATCH`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
